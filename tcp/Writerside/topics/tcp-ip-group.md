# 什么是 TCP/IP 

这篇文档主要描述了什么是 TCP/IP。从其背景到其发展历史，在到其分层实现,到其涉及的一些常见硬件，都做了简要描述。

## 什么是协议 {id="protocol"}

在生活中，我们也会有很多协议的存在。比如说平时的各类合同、借款协议等。协议约定了双方（或者更多方）的权责，换句话说规定了双方照章办事。

而计算机中的协议，和生活中的协议是一样的。协议就是一纸文本，遵循协议就是按照文本的规定去实现。

回到网络协议，其实就是国际标准化组织拟定的协议，然后所有的软硬件都按照这个协议的约定来实现对应的功能。**实现了协议，双方就可以相互理解、交流**。从这个角度来说，协议就像是双方都用同样的语言去沟通。

## 历史背景 {id="history"}

一句话总结就是，起源于美军，得益于 Unix 的实现、成就于互联网的成功，一直应用到现在。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/up12zQpbEWFKd6U9KZcm.jpg" alt="history"/>

## 设计理念 {id="design"}

David D Clark 在一篇名为 [《The Design Philosophy of The DARPA Internet Protocols》](https://dl.acm.org/doi/pdf/10.1145/52324.52336)的论文中，提出了 TCP/IP 协议设计上的七个理念:

> Internet communication must continue despite loss of networks or gateways. 

第一条说的是 TCP 协议能够容错。

> The Internet must support multiple types of communications service.

第二条说的是需要支持不同类型的通信设备(服务)，比如说思科、华为的设备。

> The Internet architecture must accommodate  a variety of networks.

第三条指的是能够连接不同类型的网络，比如说 Wifi、比如说海底光缆。


> The Internet architecture must permit distributed management of its resources.

第四条指的是，必须对资源进行分布式管理。能够有效的分配资源，因为网络带宽和处理能力是有限的，TCP/IP应该能够公平并高效地分配这些资源。

> The Internet architecture must be cost effective.

第五条指的是互联网不仅仅是技术上的创新，还需要在经济层面为各种参与者提供价值，从而确保其长期的可持续的广泛的应用。

> The Internet architecture must permit host attachment with a low level of effort.

第六条强调的是互联网的设计应该使得任何设备都可以轻松、快速且无歧视地连接到网络中去。这种开放和容易接入的特性也是 TCP 成功广泛运用的关健。

> The resources used in the internet architecture must be accountable.

第七条强调的是互联网架构中的资源管理应该是明确、透明且可追踪的，从而确保资源的公平分配、高效使用以及服务的持续优化。

## TCP/IP 协议簇 {id="tcp-ip-protocol"}

当我们说到 TCP/IP 协议的时候，要知道 TCP/IP 协议的时候。一般是说 TCP/IP 协议簇。**所谓协议簇，就是一组协议，区别于单个协议**。

那么 TCP/IP 协议簇中都包含了哪些协议呢？TCP 协议、IP 协议、HTTP 协议、SMTP 协议、POP3 协议、 Ethernet 协议、FDDI 协议、ATM 协议、PPP 协议、PPPoE 协议......

<note>
那么多的协议，我们并不需要全部记住，也不需要全部理解。我们只需要挑选其中几个重要的协议、或者开发中涉及到协议加以理解即可。比如接下去的一系列的文章，将会聚焦 TCP 协议。
</note>

## 分层结构 {id="layers"}

上面说了那么多的协议，这些协议其实是分布在 TCP 不同的层次之中的。**TCP 分为了 5 层，从上到下分别是应用层、传输层、网络层、数据链路层、物理层**。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/L1cHq903je1Eu9fDraBT.jpg" alt="layers"/>

这个层次图中，**越往上越接近用户**。下层的协议一般都内嵌在了操作系统或者硬件设备中，对于应用程序或者其用户来说是无感知的。

之所以使用分层的架构，也是为了分而治之，降低层与层之间的耦合。上层调用下层，下层支持上层，或者说这是”搭积木”。**越是下层，能实现的功能就越少、越是上层功能的实现就越丰富。**

很重要的一点是，**通过分层可以实现，不同层次之间协议的组合**。比如对于一个聊天应用程序来说，它可以选择传输层使用 TCP 协议，也可以使用 UDP 协议。

<note>
特别是在面试中，我们经常会听到 OSI 模型。它也是一种理论上的网络传输协议的模型，和 TCP 不一样的是它分的更细，有 7 层。不需要过多了解，因为它只是一种理论，在其实现之前，TCP 协议已经占据了市场，成为了业界标准。
</note>

为了知识的完整性，还是附上 OSI 参考模型和 TCP 模型的层次对应表:

<table>
<tr><td>OSI 参考模型</td><td>TCP 实现</td></tr>
<tr><td>应用层</td><td rowspan="3">应用层 </td></tr>
<tr><td>表示层</td></tr>
<tr><td>回话层</td></tr>
<tr><td>传输层</td><td>传输层</td></tr>
<tr><td>网络层</td><td>网络层</td></tr>
<tr><td>数据传输层</td><td rowspan="2">物理层 </td></tr>
<tr><td>物理层</td></tr>
</table>


## TCP 协议 {id="tcp"}

TCP（传输控制协议，Transmission Control Protocol）是一种面向连接的、可靠的、基于字节流的传输层通信协议。它是互联网协议族中的核心协议之一，常与 IP 协议（互联网协议）一起使用，被统称为 TCP/IP 协议。

其特点如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/GlDTcyeKrc6DfJoUYSXj.png" alt="TCP Futures"/>

此外，它采用的是字节流的方式，打包成报文段、保证有序接受、重复报文自动丢弃。但不维护应用报文的边界，不强制要求应用必须离散的创建数据块、不限制数据块大小。

TCP 协议为应用层提供了健壮的通信服务，比如说 HTTP（WEB服务）、SMTP（电子邮件服务）都是基于 TCP 协议的。换句话说，当你在浏览网页、发送电子邮件的时候，其实就是在使用 TCP 协议进行网络通信。

## IP 协议 {id="ip"}

这个 IP 协议，目前为止你可以理解成我们使用的 IP 地址，目前有两个版本，分别是 IPv4 和 IPv6。广泛使用的是 IPv4 版本，他是 32 位二进制组成的地址，但是它的资源面临枯竭，所以 IPv6 应运而生，它是使用 128 位二进制组成的地址。

在此不做更多介绍，后续会继续深入这个话题。

## 总结 {id="summary"}

文档重点描述了TCP/IP协议，从其起源、发展到实现的各个层次，并介绍了相关的硬件。

协议在计算机中的定义与生活中类似，是为了确保双方或多方之间的规范通信。

在网络领域，协议是按照国际标准化组织制定的规则实现的，确保各方能够互相理解和交流。TCP/IP不仅是一个单独的协议，而是一个包含多个协议（如TCP、IP、HTTP、SMTP等）的协议簇。

而TCP本身分为五个层次，从上至下分别为：应用层、传输层、网络层、数据链路层和物理层，其中越往上层功能越丰富。这种分层的架构使得不同层次的协议可以组合使用。

此外，TCP协议与OSI模型有所不同，OSI分为7层，是理论模型。

TCP作为互联网的核心协议，提供了面向连接、可靠的通信服务。

最后，IP协议有两个版本：IPv4和IPv6，其中IPv6是由于IPv4资源有限而推出的新版本。