# TCP/IP

## http协议请求过程

#### 发送方

- tcp头：协议，即选择哪个快递，申通还是顺丰
- ip头：包裹寄送地址
- mac头：目标机器mac地址，相当于收件人手机号

![](E:\images\http请求传输过程-发送.PNG)

#### 接收方

- 摘取mac头，判断是否与本机mac匹配
- 摘取ip头，判断地址，不是自己的，转发
- 摘取协议头：协议头携带端口号，根据端口号把报文给进程

![](E:\images\http请求传输过程-接收.PNG)

> mac地址与IP地址
>
> 1. mac相当于身份证，与设备绑定，ip相当于居住地址，mac与ip不是一一对应的。
> 2. 数据传输相当于寄快递，需要ip地址完成连接、数据包传输，需要mac地址确认身份

## TCP/IP

### IP

- 报文传输协议
- 实现两台主机之间点对点数据传输
- 尽力而为，报文会出现丢失、乱序、重复等

### TCP

- tcp解决报文丢失、重复发送、乱序等问题

### UDP

- 不会修复ip层产生的错误
- 将ip层**点对点**通信扩展到**进程对进程**通信

### 深入tcp/ip

- DNS解析：域名->ip

- 数据包传输：网关
- 数据包->目的地：路由协议
- 建立传输链路：握手协议

![](E:\images\tcp.png)

- 三次握手：3个数据包
  - 服务端收到SYNC后，将自己的SYNC与ACK合成了一个数据包
- 四次挥手：为什么4次？
  - 客户端不会再发送请求后，会给server端发送FIN
  - server端可能有之前的请求未处理完成，只能先ack client FIN。待自己全部处理完，再发送FIN，告诉client连接可以断开了

> UDP广播

- TCP流量控制，解决网络拥堵：滑动窗口

  - 滑动窗口大小就是缓冲区大小

  - 发送端、接收端分别维护发送窗口，接收窗口

    https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/selective-repeat-protocol/index.html

### 使用协议通信

#### socket

- socket类型与协议类型有关
  - stream socket(tcp)
  - datagram socket(udp)

![](E:\images\tcp socket.PNG)

![](E:\images\NIO.PNG)

## UDP组播协议

- 广播：网络中所有主机
- 多播：类似于发布/订阅