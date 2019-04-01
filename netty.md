# netty

## 预习

### IO模型

e.g. 网络IO

- 等待数据准备
- 将数据从内核空间复制到进程中

#### blocking IO

#### non-blocking IO

> blocking io与nonblocking io区别
>
> blocking: kernel在数据未准备好时，阻塞等待
>
> non-blocking: kernel在数据未准备好时，立刻返回错误

#### IO multiplexing

同时监控多个non-blocking io的readable事件，select readable io进行操作

#### signal driven IO

#### async IO

process发起io操作后，不等待io结果，kernel完成io后，通知process

> 同步与异步的区别：process是否等待io操作完成，才做接下来的事情

## 课程

### BIO

- 1:1模型：server接收一个请求，起一个线程处理
  - 请求量增大，线程数增大
- M:N模型：server起线程池处理请求
  - 线程池满了，后来的请求排队或被抛弃

- 缺点：
  - 创建大量线程，导致频繁的上下文切换
  - 阻塞模型，导致请求响应时间长

### NIO同步非阻塞

- NIO + 单线程reactor
- 

### UNIX网络编程5种IO模型

socket操作流程

1. 用户进程发起系统调用
2. 内核准备数据
3. os将数据从内核缓冲区cp到用户缓冲区
4. 系统调用返回结果给用户进程

#### 阻塞io

#### 非阻塞io

#### IO复用

> 同步 && 异步：用户进程在IO操作时，是否等拿到IO结果后才做下一步
>
> 阻塞 && 非阻塞：用户进程发起IO系统调用，该调用是否立刻返回

### 作业1

聊天室

- 每个客户端有唯一标识id

- server接收到client socket后，保存客户端id与socket映射关系，Map<clientId, socket>

- server建立消息队列：Map<clientId, LinkedBlockQueue<String>> messages

- 客户端发送给server消息格式

  ```json
  {
      id: clientId,
      receiver: recvId1, recvId2
      msg: msgText
  }
  ```

- 服务端接收到客户端socket以后

  - 读取socket, 解析并放入消息队列
  - 从消息队列取出需要推送给client的消息，推送给socket





## NIO

### 概念

- channel: 
  - connections to 文件，sockets
  - 支持非阻塞读
- buffer:
  - array-like
  - channel可以直接读写
- selector: Tell which of a set of Channels have IO events 
- selection key: 保存io事件状态和事件绑定

|                | NIO           | BIO                       |
| -------------- | ------------- | ------------------------- |
| ...            | SocketChannel | socket                    |
| 数据通道       | buffer        | inputStream, outputStream |
| 多路复用选择器 | selector      |                           |

### buffer读写模型

#### props

- position：初始为0，每读/写一个字节，+1
- limit：初始为capacity。读操作时，指向最后一个数据字节
- capacity
- mark: 重读标记

#### methods

- flip(): 写后读，将limit指向position，position指向0
- rewind()：读后读，position置0，limit不变，mark置-1

> read: os socket ---> memory
>
> write: memory ---> os socket

### 多路复用选择器selector

1. `selector.select()`为什么是阻塞的

   ```java
   private int lockAndDoSelect(long var1) throws IOException {
       // 对selector加对象锁
           synchronized(this) {
               .......
           }
       }
   ```

![](E:\images\nio_selector.png)



## netty

- Channel
- ByteBuf
- NioEventLoop
- ChannelHandler
- Pipeline: ChannelHandler chain

`ServerBootstrap`

mainReactor: boss thread，接收连接

subReactor: worker thread, 请求简单处理

demo: NettyServer

```java
int BIZGROUPSIZE = cpu处理器数 * 2；
int BIZTHREADSIZE = 100;

// boss, worker thread
EventLoopGroup bossGroup = new EventLoogGroup(BIZGROUPSIZE);
... workGroup = new ...(BIZTHREADSIZE);

public static void start() {
    serverBootstrap.group(bossGroup, workGroup)
        .channel(NioServerSocketChannel.class)
        .childHandler(new ChannelInitializer<Channel>(){
            
        })
}

```

- handler：处理服务端逻辑
- childHandler：处理连接



`ServerBootstrap#bind()`

-> `initAndRegister()`: 初始化&注册channel

---> `newChannel()`:

