# Netty

## NIO

### 基本概念

- channel : connection to file, socket etc. 支持异步IO操作
- buffer: array-like, channel可读写
- selector: 
  - 向selector注册channel及channel感兴趣的IO事件
  - selector.select()获取IO事件就绪的channel数量。selector.selectedKeys()获取IO就绪的selectionKeys
- selectionKeys: 包装channel, 事件，channel处理器

## Reactor模型

### NIO + 单线程Reactor模型

一个selector，所有channel注册到selector上，IO事件发生时，将channel交给不同的handler处理

- acceptor：处理ServerSocketChannel的OP_ACCEPT事件
- dispatch：将IO事件就绪的channel交给handler处理

```java
public class SingleThreadReactor implements Runnable {
    private Selector selector;
    private ServerSocketChannel serverSocketChannel;

    public SingleThreadReactor() {
        try {
            selector = Selector.open();
            serverSocketChannel = ServerSocketChannel.open();
            serverSocketChannel.bind(new InetSocketAddress(8888));
            serverSocketChannel.configureBlocking(false);
            SelectionKey sk = serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
            sk.attach(new Acceptor());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 循环监听注册的channel IO事件就绪。如果channel IO事件就绪后，将channel分发给绑定的handler处理
     */
    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                selector.select();
                Set<SelectionKey> set = selector.selectedKeys();
                Iterator<SelectionKey> iter = set.iterator();
                while (iter.hasNext()) {
                    dispatch(iter.next());
                    iter.remove();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void dispatch(SelectionKey next) {
        Runnable runnable = (Runnable) next.attachment();
        if (runnable != null) {
            runnable.run();
        }
    }

    /**
     * ServerSocketChannel OP_ACCEPT事件处理器
     */
    private class Acceptor implements Runnable {

        @Override
        public void run() {
            SocketChannel sc = serverSocketChannel.accept();
            if (sc != null) {
                // Handler: SocketChannel OP_READ | OP_WRITE事件处理器
                new Handler(selector, sc);
            }
        }
    }
    
    final class Handler implements Runnable {
        .....
    }
}
```

### 多线程变种

#### worker threads

channel完成read后，放入任务队列

worker线程池从队列取任务，并发执行

执行完成后，channel write

#### 一主多从reactor

Acceptor建立连接后，将SocketChannel分发给多个subReactor处理

## Netty

### ChannelPipeline

```java
 *                                                 I/O Request
 *                                            via {@link Channel} or
 *                                        {@link ChannelHandlerContext}
 *                                                      |
 *  +---------------------------------------------------+---------------+
 *  |                           ChannelPipeline         |               |
 *  |                                                  \|/              |
 *  |    +---------------------+            +-----------+----------+    |
 *  |    | Inbound Handler  N  |            | Outbound Handler  1  |    |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |              /|\                                  |               |
 *  |               |                                  \|/              |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |    | Inbound Handler N-1 |            | Outbound Handler  2  |    |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |              /|\                                  .               |
 *  |               .                                   .               |
 *  | ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
 *  |        [ method call]                       [method call]         |
 *  |               .                                   .               |
 *  |               .                                  \|/              |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |    | Inbound Handler  2  |            | Outbound Handler M-1 |    |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |              /|\                                  |               |
 *  |               |                                  \|/              |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |    | Inbound Handler  1  |            | Outbound Handler  M  |    |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |              /|\                                  |               |
 *  +---------------+-----------------------------------+---------------+
 *                  |                                  \|/
 *  +---------------+-----------------------------------+---------------+
 *  |               |                                   |               |
 *  |       [ Socket.read() ]                    [ Socket.write() ]     |
 *  |                                                                   |
 *  |  Netty Internal I/O Threads (Transport Implementation)            |
 *  +-------------------------------------------------------------------+
```

channel pipeline是channel独有的

channel handler可以复用

### netty与NIO关联关系

#### `NioSocketChannel`如何创建java NIO `SocketChannel`

```java
io.netty.channel.socket.nio.NioSocketChannel#NioSocketChannel()
    -> NioSocketChannel#NioSocketChannel(java.nio.channels.spi.SelectorProvider)
    	-> io.netty.channel.socket.nio.NioSocketChannel#newSocket
    	   return provider.openSocketChannel();
```

`NioSocketChannel`父类方法中设置感兴趣事件为OP_READ, `configureBlocking(false)`

#### Netty何时创建`NioSocketChannel`

```
io.netty.bootstrap.Bootstrap#connect(java.lang.String, int)
-> io.netty.bootstrap.Bootstrap#doResolveAndConnect
-> 初始化channel, 并向eventLoopGroup注册：io.netty.bootstrap.AbstractBootstrap#initAndRegister
    -> channel = channelFactory.newChannel();
    -> init(channel);
    -> selectionKey = javaChannel().register(eventLoop().unwrappedSelector(), 0, this);
-> io.netty.channel.socket.nio.NioSocketChannel#doConnect
	-> boolean connected = SocketUtils.connect(javaChannel(), remoteAddress);

```

#### client端如何从selector获取就绪的IO事件

```java
io.netty.channel.nio.NioEventLoop#run
-> select(wakenUp.getAndSet(false)); 
	-> int selectedKeys = selector.select(timeoutMillis);
-> processSelectedKeys();
处理OP_CONNECT, OP_READ, OP_WRITE:io.netty.channel.nio.NioEventLoop#processSelectedKey(java.nio.channels.SelectionKey, io.netty.channel.nio.AbstractNioChannel)
```

