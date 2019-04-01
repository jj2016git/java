# java rmi

## what?

- client调用server本地方法

## problem?

- server如何发布服务？
- client如何调用服务？

## how?

### server端

- 定义接口及实现类for远程调用

```java
public interface IHelloService extends Remote {
    String sayHello(String msg) throws RemoteException;
}

public class HelloServiceImpl extends UnicastRemoteObject implements IHelloService {
    protected HelloServiceImpl() throws RemoteException {
        super();
        // --> exportObject((Remote) this, port);
    }

    @Override
    public String sayHello(String msg) throws RemoteException {
        return "hello, " + msg;
    }
}
```

> 1. 接口需要继承``java.rmi.Remote``
>
> 1. 接口中**每个**方法都需要`throws RemoteException`
> 2. 实现类继承`UnicastRemoteObject`，并重载默认构造方法中。该构造方法会调用`java.rmi.server.UnicastRemoteObject#exportObject(java.rmi.Remote, int)`，export自身

- 创建并export远程对象
- 注册远程对象

```java
public class Server {

    public static void main(String[] args) {
        try {
            // 1. 创建并export远程对象
            IHelloService helloService = new HelloServiceImpl();

            // 2. 在server端创建注册中心
            Registry registry = LocateRegistry.createRegistry(1099);
            // 3. 注册remote object
            registry.bind("rmi://localhost:1099/Hello", helloService);
            System.out.println("server start");
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```

### client端

```java
public class Client {
    public static void main(String[] args) {
        try {
            IHelloService helloService = (IHelloService) LocateRegistry.getRegistry().lookup("rmi://localhost:1099/Hello");

            System.out.println(helloService.sayHello("client"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## why?

### 类图

![](D:\安全桌面专用目录\传文件\rmi_classes.png)

### 原理图

![](E:\images\rmi.png)

### 1. server

#### (1) exportObject `helloService`

```java
UnicastRemoteObject#exportObject(helloServiceImpl, new UnicastServerRef(port))
```

-  `unicastServerRef`引用`liveRef`,`liveRef`保存如下tcp连接信息

  **[endpoint:[99.48.24.193:0](local),objID:[39884050:166527772c7:-7fff, 11456538339601008]]**

-> `sun.rmi.server.UnicastServerRef#exportObject(helloServiceImpl, null, false)`

```java
// 为helloServiceImpl创建代理对象
Remote var5 = Util.createProxy(var1.getClass(), this.getClientRef(), this.forceStubUse);

// 将proxy包装成Target对象，将其暴露在tcp端口上，等待client调用
Target var6 = new Target(var1, this, var5, this.ref.getObjID(), var3);
this.ref.exportObject(var6);
this.hashToMethod_Map = (Map)hashToMethod_Maps.get(var4);
return var5;
```

-------------------------------------------------------------------------------------------------------------------------------------

->tcp传输层exportObject: `this.ref.exportObject(var6);`           

------> `sun.rmi.transport.tcp.TCPTransport#exportObject`

------------------>  `sun.rmi.transport.tcp.TCPTransport#listen`

(1) 创建serverSocket

(2) 创建新线程，线程AcceptLoop监听客户端请求，并处理

```java
this.server = var1.newServerSocket();
Thread var3 = (Thread)AccessController.doPrivileged(new NewThreadAction(new TCPTransport.AcceptLoop(this.server), "TCP Accept-" + var2, true));
var3.start();
```

#### (2) 启动registry服务

`sun.rmi.registry.RegistryImpl#RegistryImpl(int)`

```java
this.setup(new UnicastServerRef(var2, RegistryImpl::registryFilter));
```

**[endpoint:[99.48.24.193:1099](local),objID:[0:0:0, 0]]**

-> `sun.rmi.registry.RegistryImpl#setup`

-----> `sun.rmi.server.UnicastServerRef#exportObject(registryImpl, null, true)`

```java
public Remote exportObject(Remote var1, Object var2, boolean var3) throws RemoteException {
    Class var4 = var1.getClass(); // RegisterImpl

    Remote var5;
    try {
        // 1. 创建RegistryImpl_stub对象
        var5 = Util.createProxy(var4, this.getClientRef(), this.forceStubUse); 
    } catch (IllegalArgumentException var7) {
        throw new ExportException("remote object implements illegal remote interface", var7);
    }

    if (var5 instanceof RemoteStub) {
        // 2. 创建Registrympl_skel对象
        this.setSkeleton(var1);
    }

    Target var6 = new Target(var1, this, var5, this.ref.getObjID(), var3);
    // 3. LiveRef的exportObject方法，建立serverSocket监听，将object暴露到tcp端口上，等待client调用
    this.ref.exportObject(var6);
    this.hashToMethod_Map = (Map)hashToMethod_Maps.get(var4);
    return var5;
}
```

#### (3) registry bind remote object

- 将uri与remote object的映射关系写入map

### 2. client

#### (1) 获取registry 

`java.rmi.registry.LocateRegistry#getRegistry(java.lang.String, int, java.rmi.server.RMIClientSocketFactory)`

```java
return (Registry) Util.createProxy(RegistryImpl.class, ref, false);
```

**返回： `RegistryImpl_Stub[UnicastRef [liveRef: [endpoint:[99.48.24.193:1099](remote),objID:[0:0:0, 0]]]]`**

#### (2) 获取remote object proxy

- `sun.rmi.registry.RegistryImpl_Stub#lookup`

  -> `java.rmi.server.RemoteRef#newCall`

```java
public RemoteCall newCall(RemoteObject var1, Operation[] var2, int var3, long var4) throws RemoteException {
        // 建立client与server之间的tcp连接，连接到server RegisterImpl_skel对象
        Connection var6 = this.ref.getChannel().newConnection();

        try {
            StreamRemoteCall var7 = new StreamRemoteCall(var6, this.ref.getObjID(), var3, var4);

            try {
                this.marshalCustomCallData(var7.getOutputStream());
            } catch (IOException var9) {
                throw new MarshalException("error marshaling custom call data");
            }

            return var7;
        } catch (RemoteException var10) {
            this.ref.getChannel().free(var6, false);
            throw var10;
        }
    }
```



#### (3) proxy调用远程方法

-> `java.rmi.server.RemoteObjectInvocationHandler#invokeRemoteMethod`

```java
return ref.invoke((Remote) proxy, method, args,
                              getMethodHash(method));
```

--------> `sun.rmi.server.UnicastRef#invoke(java.rmi.Remote, java.lang.reflect.Method, java.lang.Object[], long)`

```java
public Object invoke(Remote var1, Method var2, Object[] var3, long var4) throws Exception {
    // 1. 建立tcp连接
    Connection var6 = this.ref.getChannel().newConnection();
    // 2. 执行远程调用
    StreamRemoteCall var7 = new StreamRemoteCall(var6, this.ref.getObjID(), -1, var4);
	var7.executeCall();

    // 3. 处理方法返回值
    Class var46 = var2.getReturnType();
    if(var46 != Void.TYPE) {
        var11 = var7.getInputStream();
        Object var47 = unmarshalValue(var46, (ObjectInput)var11);
        var9 = true;
        clientRefLog.log(Log.BRIEF, "free connection (reuse = true)");
        this.ref.getChannel().free(var6, true);
        Object var13 = var47;
        return var13;
    }
    var11 = null;
    return var11;
} 
```



server

HelloServiceImpl  --> Proxy[IHelloService....]
exportObject(helloServiceImpl, null, false)
--> liveRef: [endpoint:[99.48.24.193:0](local),objID:[71050d9b:16652c6237b:-7fff, 8522922440756927419]]
--> target: stub = Proxy[IHelloService,RemoteObjectInvocationHandler[UnicastRef [liveRef: [endpoint:[99.48.24.193:0](local),objID:[-f17cc4d:16652ca4cca:-7fff, 3437738127722485179]]]]]
--> server: ServerSocket[addr=0.0.0.0/0.0.0.0,localport=50726]

RegistryImpl  --> RegistryImpl_Stub
exportObject(registryImpl, null, true)
--> liveRef: [endpoint:[99.48.24.193:1099](local),objID:[0:0:0, 0]]
--> target: stub=RegistryImpl_Stub[UnicastRef [liveRef: [endpoint:[99.48.24.193:1099](local),objID:[0:0:0, 0]]]]
--> server: ServerSocket[addr=0.0.0.0/0.0.0.0,localport=1099]


client
LocateRegistry.getRegistry() ---> RegistryImpl_Stub对象
java.rmi.registry.Registry#lookup()
--> RemoteCall var2 = this.ref.newCall(this, operations, 2, 4905912898345647071L);   // client向server Target[RegistryImpl]发起tcp连接
	ref=[endpoint:[99.48.24.193:1099](remote),objID:[0:0:0, 0]]



client端方法调用过程：

- The client-side runtime opens a connection to the server using the host and port information in the remote object's stub and then serializes the call data.
- The server-side runtime accepts the incoming call, dispatches the call to the remote object, and serializes the result (the reply string "Hello, world!") to the client.
- The client-side runtime receives, deserializes, and returns the result to the caller.
