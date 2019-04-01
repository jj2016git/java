TCP

client与server之间的点对点通信

- server端：

  - 创建ServerSocket，绑定端口
  - `socket = serverSocket.accept()`接收客户端连接

- client端：

  - 创建socket，指定服务端的ip和端口

    > client socket通过随机端口访问server

- server端和client端通过socket的输入、输出流进行数据传输

UDP

广播

- server端
  - 创建DatagramSocket，绑定端口
  - `socket.receive(new DatagramPacket(recvBuf, recvBuf.length))`通过数据报接收client数据，将数据填充到`recvBuf`中
