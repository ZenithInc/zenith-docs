# 大型应用的架构概述 

在当今数字化时代，大型应用的开发与部署已成为企业成功的关键要素。这些应用承载着庞大的用户流量、复杂的业务逻辑和海量的数据处理需求。为了有效应对这些挑战，构建一个可靠、可扩展且高性能的架构显得尤为重要。

## 大型应用的特点 {id="large-scale-application-features"}

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/Uns94SESSwuKkI7E2J9o.png" alt="large scale application features"/>

1. **高并发，大流量**： 一个网站从小做到大，业务越来越大，积累的用户越来越多，网站的请求量也就越来越大。举例： 百度日均访问量达到 50亿+、腾讯QQ早就突破了 1 亿用户同时在线、淘宝双十一交易金额达到 2千多亿。
2. **高可用**： 必须保证网站稳定地向用户提供 7 * 24 小时不间断访问，服务器宕机也需要有用备用替换。
3. **大数据**： 通过大量服务器以及存储系统管理海量的数据，数据多了可以做大数据分析、用户画像、分析用户购买习惯、预测用户购买商品等等。
4. **敏捷开发、快速迭代**： 一般来说 1 到 2 周迭代一次。
5. **用户体系庞大**： 既然一个大网站的用户量是非常大的，用户可能分散在全国乃至全球。
6. **可持续升级**： 网站的演进是渐进的，不是一蹴而就的。需要随着业务的发展，不断迭代技术，而不是一上来就是高大上的设计或者面向大厂的设计。
7. **安全防范**： 要防范各种攻击，避免各种漏洞。
8. **弹性扩展**： 比如在双十一的时候，需要快速添加大量的服务器投入运行，而平时则不需要，可以随时减少。
9. **吞吐量高，响应速度快**： 一个系统数据量大了，响应速度会慢，影响用户体验。所以大型网站必须保障用户的每次请求都快速响应，背后是成百上千台服务器，但用户是不知道的。

## 设计宗旨 {id="design-purpose"}

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/mMBZKyMxMFdwezYe8lMk.png" alt="design purpose"/>

在平时的开发中，对于项目架构的设计，需要注意：

* **合久必分**：比如 MVC 架构就是系统拆分为 Model、View、Controller 三块。分层有利于代码的结偶和合理的工作划分。
* **集群**： 集群是实现高可用和负载均衡的手段，保障负载均衡的同时提升系统的可用性，相互容灾。
* **CDN**：因为大型网站的用户是分散在全国各地，乃至全球各国，所以需要采用 CDN，不管用户在哪都能访问到相对较近的节点。
* **分布式系统**： 大型网站肯定是多系统、多模块、多中间件、多服务器等协同整合的一个整体。在分布式领域中，我们会接触到分布式架构、分布式文件系统、分布式锁、分布式事务、分布式配置、分布式限流以及日记收集加工分析等。
* **异步**： 一步是最常见的优化用户体验的一种方式。比如说用 Ajax、消息队列等技术进行代码解耦。
* **业务分离**： 将一些模块进行分离，独立出一个子系统来给专门的团队负责，为其他模块提供服务。
* **数据备份**： 不能因为宕机而导致数据丢失，要定期做冷热数据备份，提供系统的高可用性。

## 大型应用的演进过程 {id="evolution-process-of-large-scale-applications"}

一开始，都是一些静态网站。网站和用户之间并没有交互，更多的是信息的输出。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/ZsamXpr6KyGVUdWHGQR8.jpg" alt="evolution-process-of-large-scale-applications"/>

到了后面，就出现了单体架构。基本上一台服务器上部署后端程序、文件服务以及数据库。用户是数量也是比较少的，所以一台服务器就可以支撑。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/pjN5sp65vmZ2PdhY6o1q.jpg" alt="evolution-process-of-large-scale-applications"/>

随着用户的数量增多，我们会把文件、数据库和业务逻辑分离。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/1PAssMcjMxt50sHTWmQy.jpg" alt="evolution-process-of-large-scale-applications"/>

这时候业务的发展就不会被单体的架构束缚，网站的承载能力有所提高。随着业务的发展，用户的查询量会越来越大，数据库的查询压力也会越来越大，然后就会加入缓存。用户绝大多数的查询都会落在缓存中，减少了数据库的压力。避免了因为数据库查询导致的性能瓶颈。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/XlGrWk1arWgvp89RQH2d.jpg" alt="evolution-process-of-large-scale-applications"/>

到目前为止，上面所有的服务都是单节点部署。就会存在单节点失效的问题，单节点失效就会导致整个服务不可用。然后就会引入集群和负载均衡:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/Lrltjre82o1pCOWu4utN.jpg" alt="evolution-process-of-large-scale-applications"/>

当网站的访问到达百万或者千万级别的时候，这样的架构已经不能满足了。数据库的读和写会成为瓶颈，这时候网站的架构演变为如下模式：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/SlAdFKaAPU5kk1yShIol.jpg" alt="evolution-process-of-large-scale-applications"/>

随着压力的再次加大，就需要将数据表以及数据库进行拆分，当单表的数据达到七八百万的时候就可以考虑这么做。这就是所谓的分库分表，架构如下:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/cZ65PZqIgU0znvNgjakM.jpg" alt="evolution-process-of-large-scale-applications"/>

再接着，会引入搜索引擎，提供更为强大的检索能力，也代替了部分关系型数据库的功能。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/L3wr9YgkM86MPXZefyGw.jpg" alt="evolution-process-of-large-scale-applications"/>

当我们的业务更加复杂的时候，就会进行业务、模块的拆分。想一个个模块拆分成一个个子系统，可以由专门的团队负责、可以独立开发、独立部署。但对于开发、测试、运维来说也都变得更为复杂，需要分布式事务的支持，优点业务分离、开发人员的职责范围更小、关注点更为集中、系统的负载能力大为增强，

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/0s6uAI1neDavlTHHUvP0.jpg" alt="evolution-process-of-large-scale-applications"/>