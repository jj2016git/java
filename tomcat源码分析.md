# tomcat源码切入点

## 手写tomcat

```java
class GPTomcat {
    private Collection<Servlet> servlets = new ArrayList<>();
    
    public void init() {
        ServerSocket serverSocket = new ServerSocket(8080);
        servlets.add(new RegistryServlet());


        
        Socket socket = serverSocket.accept();
        InputStream inputStream = socket.getInputStream();
        OutputStream outputStream = socket.getOutputStream();
        
        Request request = new Request(inputStream);
        Response response = new Response(outputStream);
        // servlet处理请求，返回回应
        Servlet servlet = getServletFrom(servlets);
        servlet.handle(request, response);
    }
}

interface Servlet {
    void handle(Request request, Response response);
}

class RegistryServlet implments Servlet {
    .......
}
```

## tomcat源码

### tomcat源码切入点1：`servlet`被加载

> 猜想一：源码中也会有一个Servlet集合，`add(servlet)`

```java
// Tomcat源码--如何收集并加载loadOnStartup的servlet
/**
     * Load and initialize all servlets marked "load on startup" in the
     * web application deployment descriptor.
     *
     * @param children Array of wrappers for all currently defined
     *  servlets (including those not declared load on startup)
     * @return <code>true</code> if load on startup was considered successful
     */
    public boolean loadOnStartup(Container children[]) {

        // Collect "load on startup" servlets that need to be initialized
        TreeMap<Integer, ArrayList<Wrapper>> map = new TreeMap<>();
        for (int i = 0; i < children.length; i++) {
            Wrapper wrapper = (Wrapper) children[i];
            int loadOnStartup = wrapper.getLoadOnStartup();
            if (loadOnStartup < 0)
                continue;
            Integer key = Integer.valueOf(loadOnStartup);
            ArrayList<Wrapper> list = map.get(key);
            if (list == null) {
                list = new ArrayList<>();
                map.put(key, list);
            }
            list.add(wrapper);
        }
    
        // Load the collected "load on startup" servlets
        for (ArrayList<Wrapper> list : map.values()) {
            for (Wrapper wrapper : list) {
                try {
                    wrapper.load();
                } catch (ServletException e) {
                   ......
                }
            }
        }
        return true;
    
    }

// map
<1, [default, dispatcherServlet]>
<3, [jsp]>
```

```xml
// servlet loadOnStartup配置
// 1. web.xml
<servlet>
    <servlet-name>default</servlet-name>
    <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
    <init-param>
        <param-name>debug</param-name>
        <param-value>0</param-value>
    </init-param>
    <init-param>
        <param-name>listings</param-name>
        <param-value>false</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet>
    <servlet-name>jsp</servlet-name>
    <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
    <init-param>
        <param-name>fork</param-name>
        <param-value>false</param-value>
    </init-param>
    <init-param>
        <param-name>xpoweredBy</param-name>
        <param-value>false</param-value>
    </init-param>
    <load-on-startup>3</load-on-startup>
</servlet>

// 2. WEB-INF/web.xml
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring/servlet-context.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

- `StandardContext#loadOnStartup()`加载servelets

> 问题：怎么找到`StandardContext#loadOnStartup()`

- `server.xml` <=> tomcat架构图 <=> tomcat源码![](https://github.com/jj2015wing/images/blob/master/tomcat_arch.jpg)

  ```xml
  <Server port="8005" shutdown="SHUTDOWN">
    <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
    <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
    <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
    <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
    <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
  
    <GlobalNamingResources>
      <Resource name="UserDatabase" auth="Container"
                type="org.apache.catalina.UserDatabase"
                description="User database that can be updated and saved"
                factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
                pathname="conf/tomcat-users.xml" />
    </GlobalNamingResources>
  
    <Service name="Catalina">
      <Connector port="8080" protocol="HTTP/1.1"
                 connectionTimeout="20000"
                 redirectPort="8443" />
      <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
      <Engine name="Catalina" defaultHost="localhost">
        <Realm className="org.apache.catalina.realm.LockOutRealm">
          <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
                 resourceName="UserDatabase"/>
        </Realm>
  
        <Host name="localhost"  appBase="webapps"
              unpackWARs="true" autoDeploy="true">
          <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                 prefix="localhost_access_log" suffix=".txt"
                 pattern="%h %l %u %t &quot;%r&quot; %s %b" />
            <context path="/demo" docBase="...">
        </Host>
      </Engine>
    </Service>
  </Server>
  
  ```


  - Context`建立web项目与url的对应关系 -> `Context代表一个web项目

> 推断：xml标签与tomcat类一一对应，e.g. <server>对应Server...

- servlet包含在Context中 -> `StandardContext#loadOnStartup()`

### tomcat源码切入点2：什么时候监听端口

- socket连接需要监听端口，图上Connector负责连接
- `Connector#startInternel`
  - `protocalHandler#start()` -> `AbstractProtocal#start()`
    - `AbstractEndPoint#start()` ->`AbstractEndPoint#bind()`
      - `NioEndpoint#bind()`->`serverSock`
      - ...

### 分析源码

![](https://github.com/jj2015wing/images/blob/master/tomcat_hirarchy.JPG)

BootStrap#main()

- `Bootstrap#init()`: 初始化类加载器，初始化Catalina对象Daemon
- `Bootstrap#load()`
  - `Catalina#load()`: 载入`server.xml`，生成server, service, engine, executor, connector, host, context等

     ```java
      Digester digester = createStartDigester(); // 配置server.xml解析及对象创建规则
      digester.parse(inputSource); // 按照文件及规则生成Object tree
     ```

    - `server#init()` -> `Service#init()` -> `Engine/Executor/Connector#init()`
- `Bootstrap#start()`:

  - `Catalina#start()`

    - `server.start()`->`service.start()`->`engine.start()`, `executor.start()`, `connector.start()`

      - `engine.start()`->`ContainerBase#startInternal()`

        ```java
        // Start our child containers, if any
        Container children[] = findChildren();
        List<Future<Void>> results = new ArrayList<>();
        for (int i = 0; i < children.length; i++) {
            results.add(startStopExecutor.submit(new StartChild(children[i])));
        }
        ```

        - host, context启动大抵如此
