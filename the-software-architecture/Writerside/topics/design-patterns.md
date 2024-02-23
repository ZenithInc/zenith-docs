# 设计模式 

设计模式这个词在软件开发中，经常指的是面向对象编程中 23 中设计模式。这篇文档列举了一些在微服务架构设计中常见的模式，介绍了这些模式的优缺点。了解了这些模式之后，对微服务的设计会有整体的认识。

## 微服务设计模式 {id="top-10-microservice-design-patterns"}

这篇文档中所列举的模式，也是在微服务实践中被广泛应用的一些通行的做法，下面列举了不同类目下的设计模式。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/ox352IvFAHEUtJgB0uer.png" alt="the microservice design patterns"/>

所谓的模式，有时候也可以理解成是某种验证过的“做法”。比如 Database per Service 指的是每一个微服务都应该有自己的数据库。这也符合“**通过接口来通信，而不是通过数据共享来通信**”的原则。有模式，自然有反模式。

下文将选择其中的一些模式来讲解。

## 数据共享？ {id="data-share-pattern"}

但是有些老项目微服务改造中，也可以采用这种做法，不必要一次性把数据库都分离开来，只要数据库能承受住压力即可。这也被称之为“数据共享模式”：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/ZM7gL0mXsMDGtXDki4UF.png" alt="data share pattern"/>

但是随着微服务改造的升入，这种做法势必要不能长久使用。演变成下面这种模式 Database Per Service：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/5Ru30NG61gVtykAdbMFf.png" alt="Database Per Service"/>



## 相关文章 {id="related-articles"}

1. [《Top 10 Microservice Architecture Design Patterns Every Developer Should Learn》](https://medium.com/javarevisited/top-10-microservice-design-patterns-for-experienced-developers-f4f5f782810e)
2. [《Pattern: Microservice Architecture Context》](https://microservices.io/patterns/microservices.html)
3. [《Microservice Architecture and Design Patterns for Microservices》](https://medium.com/@madhukaudantha/microservice-architecture-and-design-patterns-for-microservices-e0e5013fd58a)