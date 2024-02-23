# TCP 报文 

在 TCP 中，报文（Segment) 指的是 TCP 传输的单元。这篇文档介绍了关于它的细节，包括报文的传输、格式等。

## 报文头部的层层组装和卸载 {id="encapsulation-and-decapsulation-of-tcp-headers"}

下面这张图展示了一个报文是如何从源计算机的应用传输到目标计算机的应用的。换句话说，一条微信消息是如何经过应用程序、操作系统、网线、路由器、交换机到达另一个手机上的微信的。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/Y4GmCepACiteKd7P4WsE.jpg" alt="tcp-segment-transfer"/>

在源主机中，由应用层构造好消息，通过传输层时会加上自己的头部、经过网络层的时候继续加上自己的头部信息，之后数据链路层也是。

在经过路由器时，路由器会剥离数据链添加的头部，解析网络层的头部，在知道了要发给谁之后，会重新经过数据链路层、物理层添加头部信息，然后发送给交换机。

交换机接收到消息之后，继续卸载头部信息后拿到自己所需的信息，然后重新组装报文头部，发给目标主机。

目标主机，又层层卸载，最终将得到的数据由操作系统发送给目标主机上的应用程序。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/oeY8Oo9PYW6b1YoJ93SN.png" alt="Protocols at different Layers"/>

## 报文的格式 {id="tcp-segment-format"}

### IPv4的头部 {id="IP-Header"}

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/4Lkfi8TRQIcrMre5l9G0.png" alt="IPv4-Header"/>

### TCP的头部 {id="TCP-Header"}

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/HOlmQaZJ95gHnzhqpApW.png" alt="TCP-Header"/>

## 参考资料

1. [TCP/IP-REF](https://nmap.org/book/tcpip-ref.html)