# 序列化 and 反序列化

## what and why?

- 网络传输：对象 -> 数据流 -> 对象
- 对象持久化存储

## how?

- java: Serializable + object stream

  - 数据大

  - 语言限制

  - serialVersionUID：保证序列化与反序列化用的同一个uid。如果uid发生变化，反序列化失败

    > serverA -> serverB：
    >
    > 1. serialVersionUID发生变化，反序列化失败
    > 2. serialVersionUID不变，serverA端对象发生变化，serverB端反序列化成功
    > 3. 不设置serialVersionUID，serialVersionUID=hash code, serverA端对象发生变化，serverB端反序列化失败？？？？
    >
    > 静态变量不参与序列化
    >
    > transient变量不参与序列化
    >
    > 父子类序列化
    >
    > 手工读写流可以绕过transient
    >
    > 同一个对象序列化到同一个流两次=序列化1次 + 引用

- xml(soap)

- json

- protobufs

- messagepack

## 常见的序列化技术

hessian2

### Protobuf

- 独立语言、独立平台

- 空间开销小、性能好
- 学习成本高
- 生成的字节码看不懂

#### how?

- 下载编译器protoc.exe
- 编写独立的proto文件
- 编译

#### why?

- 位运算
  - varint做编码
    - 类似于utf8
    - 负数 zigzag->无符号整数->varint编码

  ![](E:\images\zigzag.png)

  - T-L-V做存储

## 序列化算法选型

- 序列化存储开销
- 序列化计算开销
- 平台兼容性
- 学习成本



