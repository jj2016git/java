## java

## 异常

### 层次性

- Throwable
  - checked Exception
  - unchecked Exception: RuntimeException
  - Error

### 传播性

- jdk 1.4 
  - `StackTraceElement`记录异常堆栈信息
  - `Throwable#fillInStackTrace()`
- jdk 9 `StackWalker`优化`StackTraceElement`
- 栈深度过深，导致性能消耗大
- 解决方式：
  - JVM参数，设置栈最大深度
  - logback限制异常堆栈深度

### 泛型

### type erasure

- def: 
  - 编译期，强制进行类型限制
  - 运行期，丢弃类型信息
    - 如果没有类型上限，T变成Object类型
      - 如果`T extends SuperClazz`，T变成`SuperClazz`类型
- 由编译器通过编译错误保证类型安全

#### 多态场景下，类型擦除导致问题

```java
public class Stack<E> {
    private E[] elements;
    
    public void push(E ele) {
        ...
    }
    
    public E pop() {
        ...
    }
} 

public class IntegerStack extends Stack<Integer> {
    public void push(Integer ele) {
        ...
    }
}

// demo
IntegerStack is = new IntegerStack(5);
Stack s = is;
s.push("hello");
Integer data = is.pop(); //ambigurity
```

```java
// Stack类型擦除后
public class Stack {
    public void push(Object ele) {
        ...
    }
}
```

- jvm解决方案：bridge methods

  ```java
  public class IntegerStack extends Stack {
      // 编译器自动生成的bridge method, 该方法内部完成Object->指定类型的转换，并调用用户类的方法实现功能
      public void push(Object value) {
          push((Intger) value);
      }
      
      // 用户定义的方法
      public void push(Integer value) {
          ...
      }
  }
  ```


## 函数式编程

### 匿名内置类





## java接口设计

### 通用设计

#### 类名称：（形容词） + 1-N个名词

#### 可访问性

- public
- default(package)：包内访问权限，私有api, e.g. FileSystem, FinalReference
- protected
- private

> protected, private不能修饰最外层class

> 《Practical api design》

#### 可继承性

- final：e.g. java.lang.String

  - String为什么设置为final

    ```java
    // 对象类型常量化
    // 语法特性：java对String提供类原生类型的支持
    String value="aa";
    ```

  - String可以通过反射修改

- 非final：FileSystem

### 具体类设计

#### 抽象类设计

- 常见场景
  - 模板模式：e.g. AbstractMap/Set/List
  - 状态、行为继承
- 常见模式
  - 介于类和接口之间（可由接口default方法替代）
  - Abstract/Base前缀

### 接口设计

- 场景：

  - 通信契约

    - api
    - rpc

  - 常量定义（已被枚举取代）

    ```java
    interface Constant {
        int VALUE = 0;
    }
    ```

- 接口编程

  - 强类型约束

- 标记接口（没有方法）

  - Serializable
  - Cloneable
  - AutoClosable
  - Listener

- 常见模式

  - 无状态

### 内置类设计

> 《Effective java》

- 常见场景

  - 临时数据存储，e.g. ThreadLocal$ThreadLocalMap

  - 特殊用途的api实现：e.g. Collections$UnmodifiableCollection
  - Builder模式（接口）：Stream$Builder

- 访问限制

## java枚举设计

### 枚举类

- 相对于常量，有强类型约束
- final类

### 枚举

- 强类型约束
- 继承java.lang.Enum
  - Enum默认属性为name, ordinal
- 不可显式的继承与被继承

> `values()`是java编译器给枚举提升的方法
>
> 枚举实质上是final class, 成员修饰符为public static final
>
> 枚举可以实现接口

> 枚举内部可以定义抽象方法，e.g. TimeUnit

> 字节码提升：《java字节码规范》《深入java虚拟机》classLoader部分

> interface为什么引入default方法，interface如果增加方法的话，所有实现类均需要实现该方法。为了接口升级不影响实现类，添加default方法，实现类可以有选择的override default方法

## java泛型设计

### 泛型使用场景

- 编译时强类型检查
- 避免类型强转
- 实现通用算法

### 泛型类型

#### 类型参数命名规定

- T: type, 多于1个：U, R...
- K, V: key, value
- E: element

#### bounded类型参数

##### 单一界限

- 上限：`<T extends SuperClass>`
- 下限：`<T supers SubClass>`

##### 多重界限

`<T extends Class1/Interface1 & Interface2 & ....>`

- 第一个上限是class或接口
- 后面必须是接口

> 遵从java多重继承规范，只能继承一个class, 可以实现多个接口
>
> === clazz extends Class1 implements Interface1, ...

```java
        // new Container(...)这种写法等同于new Container<Number>(...)
        Container<Integer> intCon = new Container(1.0f);
        // 编译错误，diamond写法new Container<>(...)会令java compiler自动根据变量声明推断出泛型真正类型
//        Container<Integer> intCon2 = new Container<>(1.0f);
        // 编译错误，String不是Number的子类
//        Container<Integer> intCon3 = new Container("aaa");
        // 运行时，E类型擦写为Number
        System.out.println(intCon.getElement());
```



##### 通配符

- 有上限通配符
- 有下限通配符
- 完全通配符

















































# 面试案例剖析

## target

1. 如何设计分布式调度框架
2. 怎么分析面试技术点

### 案例

#### 1. 设计并实现一个高性能分布式(AP)调度框架

- 要求
  - 分布式
  - 高性能
    - cap：一致性、可用性、分区容错性，cp or ap
    - base：
      - 基本可用
      - 软状态
      - 最终一致性
  - 调度框架
    - cron表达式
    - 自运维、柔性、资源问题
- 设计思路
  - 高性能分布式=>CAP/Base=>zk/etcd实现协调服务=>ZAB/raft协议
  - master-worker架构/无中心化架构
  - 任务调度
  - 任务分发
  - 服务注册与发现
  - 其他
    - 分布式存储
    - 异步日志
    - 并发设计
    - 事件广播
    - nginx负载
- 扩展问题
  - 如果不采用分布式锁，能不能采取其他方案替代
  - raft, zab如何保证base
  - worker热点问题
  - 任务增加时，worker如何动态扩展

## 如何准备面试

1. 算法：**刷题**
2. 集合类：**使用+源码**
3.  J.U.C：**使用+源码+AQS，Synchronized、轻量级锁、偏向锁实现原理**
   1. 重量级锁、轻量级锁、偏向锁、自旋锁
      - 重量级锁实现原理 -> JVM -> OS
      - 锁升级、降级
   2. 工具类
      - aqs综合案例
4. JVM：一般不会问有哪些GC
   1. 案例：优化 + 原理
   2. CMS实现原理，G1
   3. 内存布局：调优
   4. GCROOT：虚拟机栈为何会成为GCROOT
   5. 类加载器

> 串：CMS与ConcurrentHashMap关系

5. 数据库：
   1. mysql调优思路
   2. mysql体系架构及每一块功能
   3. 执行计划怎么看
   4. 案例：现场调优
   5. 为什么mysql联合索引遵循最左匹配原则
   6. 索引的本质是什么？
      - B+树，为什么采用B+树
      - B+树为什么能减少磁盘IO读写次数
      - 有没有比B+树更好的方式
   7. 优化器工作原理
6. NIO, Netty
   1. NIO模型，UNIX 5种网络模型
   2. 手写一个局域网搜索机器的demo
   3. NIO、Netty的优化点
7. spring boot, spring cloud
   1. IOC实现
   2. 容器创建过程
8. 分布式架构：案例
9. 谈项目：抓住技术点深入分析
10. 项目管理、笔试
11. java异常处理过程及原理
12. java反射机制到源码级别
13. 设计模式

### 经典面试题

- 1 TB数据文件，TopN
- 设计一个正则表达式引擎



