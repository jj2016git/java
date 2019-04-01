维基百科：software design pattern

<https://en.wikipedia.org/wiki/Software_design_pattern>

- 面向对象23种设计模式

- 创建型模式

  - object pool: `apache commons pool`, tomcat DBCP
  - 资源获取及初始化

- 结构性模式

  - adapter, wrapper, translator

  - extension object

  - front controller: `DispatcherServlet`

  - marker: zuul，空接口关联class metadata

    ![](C:\Users\lacri\Pictures\marker_pattern.PNG)

## spring中的行为模式

### 责任链

- java FileFilter, FilenameFilter

  ```java
  File#listFiles(FilenameFilter ...)
  ```

- servlet API:     filter chain: filter -> filter ->filter

- SpringMvc:  `HandlerMapping`(请求来源)，`HandlerExecutionChain`

  -   `HandlerInterceptor`
  - `MappedInterceptor`

  ```java
  class AbstractHandlerMapping {
      List<HandlerInterceptor> ...
  }
  ```

- webflux:  `WebFilter`

### Command

- `RestTemplate#execute(...)`

### Interceptor

- `HandleInterceptor`
- Spring AOP
  - spring cache: `CacheInterceptor`
  - spring transaction ->  @Transactional  ->`TransactionInterceptor`

> 注解与拦截器关系？？

### 迭代器

- spring environment: `PropertySources`

### Mediator

mediator封装一大堆对象的交互流程

- springmvc `DispatcherServlet`

### 备忘录

- spring transaction -> JDBC特性

  - commit() | rollback() | savepoint

    ```python
    savepoint s1
    	savepoint s2
        commit(s2) || rollback(s2)
        
        savepoint s3
        commit(s3) || rollback(s3)
    commit(s1) || rollback(s1) 
    ```

### 观察者/订阅发布

- spring event/listener

  -> ApplicationEvent /ApplicationListener

- spring Lifecycle 广播

  -> callback所有实现Lifecycle的bean

  context.start() 发lifecycle, 发startEvent

### 状态模式

- AbstractApplicationContext通过实现Lifecycle实现context状态控制

### 策略

- spring ->`InstantiationStrategy`
- springmvc -> `ContentNegotiationStrategy`

### 模板方法

- spring core -> `AbstractApplicationContext`

### 访问者

- spring core ASM扩展

  - ClassVisitor
  - MethodVisitor
  - FieldVisitor

  > 实现@Component派生性

### 抽象工厂

- `factory-method`
- `factory-bean`
- `FactoryBean#getObject()`
- `ObejctFactory#getObject()`

### builder

- `BeanDefinitionBuilder`

### 工厂方法





