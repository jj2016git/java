# JVM

## what?

### 定义

Foo.java -> Foo.class --(JVM)--> machine code

**JVM屏蔽底层硬件、指令细节**，write once run everywhere

- 不同平台不同jvm

jdk, jre, jvm

![](E:\images\java_conception.PNG)

## why?为什么学习JVM

> 了解细节，处理内存泄漏

### JVM运行期数据区

https://blog.csdn.net/bruce128/article/details/79357870

#### 指令

##### 程序计数器

- 指向当前线程正在执行的字节码指令地址（行号）
- **属于线程的**
- 作用：CPU时间片切换，线程再次获取执行权时，根据程序计数器计算下一条指令地址

##### 虚拟机栈

- 存储当前线程运行方法时所需要的数据、指令、返回地址
- FILO: 栈帧先进后出
- 栈帧：线程执行一个方法时，向虚拟机栈压入一个栈帧

###### 栈帧

- 局部变量表：编译器可确认size
  - 栈指向堆
    - 指向变量堆内存地址
    - 指向句柄池句柄，句柄指向变量堆内存地址
- 操作数栈：
- 动态链接：实现动态特性，e.g. 运行时多态

> Q: 动态链接为什么在栈帧里面？
>
> A: 只有方法执行时才需要动态连接

- 出口

##### 本地方法栈

> 线程独享程序计数器、虚拟机栈、本地方法栈（for native method）

#### 数据

- 方法区：
  - 类信息、静态变量、常量（1.7有变化）、JIT（1.7以前）

> 1.7以后，字符串常量从方法区->堆

## JVM垃圾回收

### 分代

#### <1.8分代

新生代（堆）

老年代（堆）

永久代（方法区）

#### >=1.8分代

永久代->meta space

meta space相当于ArrayList，可扩容，解决永久代溢出

> meta space必须限制最大值，不然可能导致outOfMemory

### spring bean如何被回收的

- singleton：容器销毁的时候被回收
- 如何让一个对象
- prototype：

#### 什么对象可以被回收

 GC ROOT

- 什么对象可以成为GC root
- 为什么？

#### 垃圾回收器

#### GC log

#### 如何用MAT分析dump文件，定位内存泄漏

- eclipse memory analyzer

