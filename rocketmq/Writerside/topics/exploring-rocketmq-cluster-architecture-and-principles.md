# RocketMQ 集群架构和原理解析 

这篇文档介绍了 RocketMQ，其优点和能力、集群架构模型。

### RocketMQ 简介 {id="introduction"}

RocketMQ 是一款分布式队列模型的消息中间件，由阿里巴巴自主研发的一款适用于高并发、高可靠性、海量数据场景的消息中间件。早期开源的 2.x 版本名为 MetaQ。2015年迭代了 3.x 版本，更名为 RocketMQ，2016 年贡献到 Apache, 经过 1 年多的孵化，最终成为 Apache 顶级的开源项目，更新非常频繁，社区活跃度非常高。

RocketMQ 借鉴了 Apache Kafka，其消息的路由、存储、集群划分都借鉴了 Kafka 优秀的设计思路，并结合自身双十一场景进行了合理的扩展，丰富了 API。

### 特性和能力 {id="features"}

RocketMQ 支持下面这些特性和能力:

- 支持集群模式、负载均衡、水平扩展能力
- 亿级别的消息堆积能力
- 采用零拷贝的原理、顺序写盘、随机读（索引文件）
- 丰富的 API
- 代码优秀，底层通信框架使用 Netty NIO 框架
- NameServer 代替了 Zookeeper
- 强调集群无单点、可扩展，任意一点高可用，水平可扩展
- 消息失败重试机制，消息可查询
- 开源社区活跃度高
- 经过双十一考验，非常成熟

### 核心概念 {id="core-concepts"}

在 RocketMQ 中，由以下这些核心概念，需要理解:

| 概念             | 描述                               |
|----------------|----------------------------------|
| Producer       | 消息生产者，负责产生消息，一般由业务系统负责产生消息       |
| Consumer       | 消息消费者，负责消费消息，一般是后台系统负责异步消费       |
| Push Consumer  | Consumer 的一种，需要向 Consumer 对象注册监听 |
| Pull Consumer  | Consumer 的一种，需要主动请求 Broker 拉取消息  |
| Producer Group | 生产者集合，一般用于发送一类消息                 |
| Gonsumer Group | 消费者集合，一般用于接收一类消息进行消费             |
| Broker         | MQ 消息服务（中转角色，用于消息存储和产生消息转发）      |

### RocketMQ 核心源码包 {id="source-package-structure"}

RocketMQ 的一些核心源码包的说明，如下表所示:

| 包名                     | 描述                           |
|------------------------|------------------------------|
| rocketmq-broker        | 主要的业务逻辑，消息收发，主从同步，pagecache  |
| rocketmq-client        | 客户端接口，比如生产者和消费者              |
| rocketmq-common        | 公用数据结构等                      |
| rocketmq-distribution  | 编译模块，编译输出等                   |
| rocketmq-example       | 代码示例，比如生产者和消费者               |
| rocketmq-filter        | 进行 Broker 过滤不感兴趣的消息传输，减少带宽压力 |
| rocketmq-logappender   | 日志相关                         |
| rocketmq-logging       |                              |
| rocketmq-namesrv       | Namesrv服务，用于服务协调             |
| rocketmq-openmessaging | 对外提供服务                       |
| rocketmq-remoting      | 远程调用接口，封装 netty 底层通信         |
| rocketmq-srvutil       | 提供一些公用的工具方法，比如解析命令行参数        |
| rocketmq-store         | 消息存储核心包                      |
| rocketmq-test          | 提供一些测试代码包                    |
| rocketmq-tools         | 管理工具，比如有名的 mqadmin 工具        |

### Rocket 集群架构模型 {id="cluster-model"}

RocketMQ 提供了丰富的集群架构模型，包括但单点模式、主从模式、双主模式，以及生产环境使用最多的双主双从模式，下面是双主双从（多主多从）模式的架构图:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/0RDOpl1Aq2P92U7FSnXv.jpeg)

Producer 集群就是生产者集群，在同一个生产者组 Producer Croup 中。而 Consumer 集群就是消费者集群，也在同一个消费者组 Consumer Group 中。

NameServer 集群作为轻量级配置中心，只做集群元数据存储和心跳工作，不必保障节点间数据强一致性，也就是说 NameServer 集群是一个多机热备的概念。

对于 Broker 而言，通常 Master 和 Slave 为一组服务，他们互为主从节点，通过 NameServer 与外部的 Client 端暴露统一的集群入库哦。Broker 就是消息存储的核心 MQ 服务了。

### 集群架构说明 {id="cluster-remark"}

RocketMQ 作为国内顶级的消费中间件，其性能主要依赖于天然的分布式 Topic 和 Queue，并且其内存和磁盘都会存储消息数据，借鉴了 Kafka 的空中接力概念。所谓空中接力，指的是数据不一定要落地，RocketMQ 提供了同步、异步双写，同步异步复制的特性。在真实的生产环境中应该选择符合自己业务的配置。

RocketMQ 在实际生产环境中，可以采用 8M8S 的集群架构(Master-Slave), 硬件 Master 为 32C96M500SSD。其主要性能瓶颈最终会落在 IOPS 上面，当高峰来临的时候，磁盘读写能力是主要的性能瓶颈，每秒手法消息 IOPS 达到 10W+ 消息，这也是公司内部主要的可靠性消息中间件。

之所以瓶颈在 IOPS 根本原因是因为云环境导致的问题，云环境中的 SSD 物理存储显然和自建机房的 SSD 会有不小的差距。这一点我们无论是从数据库磁盘性能，还是 ElasticSearch 的磁盘性能，都能给出准确的瓶颈点，单机 IOPS 达到 1W 左右的云存储的 SSD 的性能瓶颈，这样解释了木桶短板效应。在真实的生产中，CPU 的工作主要在等待 IO 操作，高并发下 CPU 的资源接近极限，但是 IOPS 还是达不到想要的效果。

在很多时候，我们的业务会有一些非核心的消息投递，后续会进行消息中间件的业务拆分，把不重要的消息采用 Kafka 的异步发送机制，借助 Kafka 强大的吞吐量和消息堆积能力来做业务的分流。