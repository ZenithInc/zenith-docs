# 单体应用到分布式应用

随着我们国家互联网的兴起，应用的规模已经复杂程度都日益增加。从早期的 Web 到移动应用的蓬勃发展、从门户网站到社交媒体以及自媒体的发展、从单体架构走向分布式架构。

## 单体应用的优势和劣势

这里讨论单体应用的优势和劣势，并不是说用或者不用单体应用。实际上这是要考虑项目的规模和阶段的，在用户量也业务规模都不大的情况下采用分布式应用是不对的。应该用演进的架构思维去看。

先说单体应用面临的问题和挑战:

* 代码膨胀、研发成本变高、难以实现敏捷交付
* 代码维护成本变高，开发人员交接困难
* 测试成本变高，回归的工作量大，给测试带来巨大工作量
* 可扩展性差，技术升级时需要考虑整体，无法单独调整
* 越来越多的人共同在上面协同工作，可能会相互妨碍，带来所有权边界的混乱。这个问题称之为交付摩擦（Delivery contention）

但是单体架构也具备一些优势：

* 更简单的部署拓扑可以避免很多分布式系统相关的陷阱
* 可以更简单的实现监控、故障排除、端到端测试
* 前期的开发成本低、开发周期短

## 单体架构分而治之

当应用刚起步、流量不大的时候，单台服务器就可以满足。比如经典的 LAMP 架构，如下图所示:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/K1DbYuPoMdOQip7Jzw0J.svg)

这种架构的优势是开发成本低、开发周期短。但是当访问量的增加，带来更多的压力的时候，我们就会开始增加配置。但是配置并不是无上限的，并且会带来成本的成本增长。这时候，我们会采用横向扩展。将一台服务器分成多台，如下图所示:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/u6IY3lI6KWQbMBNnEwsd.svg)

在这样的架构下，不同的服务器可以有不同的配置。比如相比应用服务器，数据库所在的服务器可以有更快的磁盘 I/O，更大的内存，好让更多的热点数据都在内存中得到缓存。因为在这个阶段，压力通常是在数据库，而不是应用本身。

但是随着访问量进一步加大，我们就需要引入缓存来分担数据库的压力:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/dc5pRm6hvjRAhxzLjXm1.svg)

这种架构下，我们先读写缓存。比如微博的点赞数，我们可以先写入缓存，在缓存中递增。然后没一千次，写入数据库一次。这样对数据库写的压力就见笑了 1000 倍。读也是一样的道理，先从缓存中读取微博列表，如果缓存失效，则再从数据库中读并写入到缓存。当然实际架构可能更复杂，需要异步从数据库中读取并写入到缓存。

## 通过集群化提升并发能力

当用户进一步增长，单个服务器就很难承载更多流量。

以 Tomcat 服务器来说，在 Tomcat8 以前，每个请求都会启动一个线程来处理，导致并发请求数很难超过 1K。之后采用了 NIO 的方式来处理请求，并发请求数大大提升，即使如此也很难突破 4K。

这时候就需要对单个应用进行集群化来提升并发能力:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/gM9r1U84ROQipkMRSaf4.svg)

负载均衡器可以是软件也可以是硬件件也可以是硬件，硬件主要是 F5, 而软件有 Apache、Nginx、HaProxy、LVS。

对于其他无状态的服务也是一样的，比如多部署一些 PHP/Java 的实例服务器，然后通过 Apache/Nginx 反向代理请求给这些实例。其调度算法可以是轮询、权重、Hash 等。

## 数据库读写分离

接下来，可能的瓶颈可能会出现在数据库。按照横向扩展的思路，利用数据库自身的主从复制的机制，搭建主库和从库（一般是一主多从的拓扑），进行读写分离。这适用于读多写少的绝大部分场景。

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/FIGolmYclcTe1iDOJJ3w.svg)

应用程序也需要随之改造，从单一数据源改成多数据源，并且区分读写操作。后期，也可以加入读写分离的中间件，应用程序不再负责区分读写操作。比如 Apache ShardingSphere，除了支持读写分离，也支持分库分表的能力。

## CDN 静态资源分发网络

接着，我们需要对静态资源的访问速度通过引入 CDN 的方式进行优化。CDN 是内容分发网络，可以让用户就近访问应用的静态资源。其实就是将静态资源缓存到全国乃至世界各地的静态资源服务器上，用户访问的时候优先访问当地或者距离更近的服务。

![what is a cdn distributed server map](http://file-linker.oss-cn-hangzhou.aliyuncs.com/ndMnFlAxh6MIxwVQNjB0.png)

通过将静态资源提前缓存到各地的边缘服务器，可以分摊主服务器的带宽压力，提升用户的访问速度，也可以有效降低 DDoS 攻击的安全风险。

## 分布式文件存储系统

在我们的应用中，经常需要存储大量的文件。这时候就可以引入分布式文件存储系统，对于小文件来说，可以选择 FastDFS、TFS，对于大文件来说可以选择 HDFS。当然，也可以选择一些云产品，比如阿里云、七牛云的 OSS。例如阿里云的 OSS 架构如下:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/e6puEgBDfVOy3awdq96K.png)

## 海量数据检索和数据异构

为了支持海量的数据检索以及数据异构，我们还需要引入 NoSQL 以及数据检索引擎。像 NoSQL 比如 Redis、MongoDB、IndexDB、ElasticSearch 等。而数据检索引擎可以使用 ElasticSearch、Solr、Lucene。前两者都是基于 Lucene，可以独立部署，基于 HTTP 请求检索数据。

## 总结

本文概括了从单体应用到分布式应用的演进过程，强调了在互联网技术发展和应用规模扩大的背景下，如何通过技术和架构的迭代来适应不断增长的用户需求和系统负载。文档从以下几个关键的技术转变和优化策略进行了讨论：

1. **单体架构的扩展**：起初，应用可以部署在单台服务器上，使用如LAMP这样的经典架构。随着访问量增加，首先尝试增加服务器的配置（垂直扩展），然后通过横向扩展（增加服务器数量）来分担负载。随着负载进一步增加，引入缓存系统来减轻数据库的压力。

2. **通过集群化提升并发能力**：为了应对更大的用户量，对单个应用进行集群化，使用负载均衡器（可以是软件如Nginx，或是硬件如F5）来分发请求，提高系统的并发处理能力。

3. **数据库读写分离**：随着数据库成为潜在的瓶颈，采用主从复制机制进行读写分离，适用于读多写少的场景，进一步提高数据库的处理能力。

4. **CDN静态资源分发网络**：引入CDN来优化静态资源的访问速度，通过全国或全球的边缘服务器缓存静态资源，降低主服务器的带宽压力，提高用户访问速度。

5. **分布式文件存储系统**：对于需要存储大量文件的应用，引入分布式文件存储系统，如FastDFS、TFS（针对小文件）或HDFS（针对大文件），以及利用云服务提供的解决方案如阿里云的OSS。

6. **海量数据检索和数据异构**：为了支持海量数据的检索及处理，引入NoSQL数据库和数据检索引擎，如Redis、MongoDB、ElasticSearch等，以提高数据处理的效率和灵活性。

本文通过这一系列的技术和架构演进，展示了如何从简单的单体应用架构，逐步过渡到能够支持大规模用户和数据的分布式应用架构。
