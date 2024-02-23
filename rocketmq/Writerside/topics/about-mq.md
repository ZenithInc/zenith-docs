# 关于消息队列 

这篇文档介绍了什么是消息队列，为什么需要消息队列，已经消息队列如何选型等内容。希望通过这篇文档，能帮助你认识理解消息队列。

## 什么是消息队列 {id="what-is-mq"}

所谓中间件，就是处于多个应用中间的软件系统。而消息中间件，值得是位于多个应用之间用来处理应用产生的消息的软件系统。它主要完成三个功能，消息的收集、存储、转发。

和数据库不同，数据库是数据最后的归宿。而消息中间件，只负责消息的收集、临时存储以及转发。

## 消息队列的作用 {id="mq-use-case"}

我们以实际的场景为例，消息队列能做什么？主要有三大应用场景:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/gJIufA5hI6y9DGMBIShR.png" alt="use case"/>

### 异步处理 {id="async-tasks"}

首先，消息队列可以用来做**异步处理**。比如说秒杀的场景，我们需要完成如下内容：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/FLX4oUW5RdZh944YNvg6.png" alt="steps"/>

其实，只要风控和减库存通过了，这次秒杀就算是成功了。所以像生成订单、短信通知、更新统计数据这些事情都可以通过发送到消息队列，异步完成。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/7mF9sywN5NF8BL1Eew9P.png" alt="step2" />

这样做的好处如下:

* 可以更快地返回结果
* 减少等待，实现了步骤之间的并发，提升了系统总体性能

### 流量控制 {id="control"}

同样是秒杀的场景，我们的服务承受能力是有限的，有时候用户的流量是非预期的。当用户流量超过了我们的应用本身承受能力，系统就可能奔溃。这时候，我们可以使用消息队列来控制流量。具体有两种常见的做法:

* 全异步处理
* 令牌桶

全异步处理的方式，即当网关收到用户请求后，就将消息发送到 MQ 中。然后等待消费完成之后，应用通过 RPC 返回响应，并返还给客户端。

但这种做法对系统的侵入性大，改造成本高，改造复杂。相对简单的做法是使用令牌桶的做法。

令牌桶的做法是由令牌生成器但固定的速率生成令牌插入消息队列，应用收到请求就从消息队列中获取令牌，获取不到，就返回失败。

### 服务解耦 {id="service-decoupling"}

比如说外卖订单系统，当客户端产生了一个订单之后，还需要通知商家、骑手等其他系统。如果把代码都堆叠在一起，那么订单的逻辑会越来越庞大、复杂，响应会越来越慢。

可以通过消息队列来解耦，当处理完订单之后，将订单的消息推送到消息队列。而骑手、商家都可以通过消息队列获得完整的订单的信息，然后自行处理相关的业务即可。这就完成了订单和骑手、商家的松耦合。

## 消息队列带来的问题 {id="mq-problems"}

在计算机架构中，是不存在银弹的。消息队列也会带来一些额外的问题，需要我们考虑:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/ke3kSwyBbjN4bzYVvHlL.png" alt="problems"/>

## 消息队列的选型 {id="technology-selection"}

下面的表格给出了常见的 MQ 的特性对比:

|Message Product|Client SDK|Protocol And Specification|Ordered Message|Scheduled Message|Batched Message|BroadCast Message|Message Filter|Server Triggered Redelivery|Message Storage|Message Retroactive|Message Priority|High Availability and Failover|Message Track|Configuration|Management and Operation Tools|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|ActiveMQ|Java,NET,C++|Push model,support OpenWire, STOMP,AMQP, MQTT,JMS|ExclusiveConsumer or Exclusive Queues can ensure ordering|Supported|Not Supported|Supported|Supported|Not Supported|Supports very fast persistence using JDBC along with a high performance journal,such as levelDB,kahaDB|Supported|Supported|Supported, depending on storage,if using levelDB it requires a ZooKeeper server|Not Supported|The default configuration is low level, user need to optimize the configuration parameters|Supported|
|Kafka|Java,Scala|Pull model, support TCP|Ensure ordering of messages within a partition|Not Supported|Supported,with async producer|Not Supported|Supported,you can use Kafka Streams to filter messages|Not Supported|High performance file storage|Supported offset indicate|Not Supported|Supported, requires a ZooKeeper server|Not Supported|Kafka uses key-value pairs format for configuration. These values can be supplied either from a file or programmatically.|Supported, use terminal command to expose core metrics|
|RocketMQ|Java,C++|Pull model,support TCP,JMS,OpenMessaging|Ensure strict ordering of messages,and can scale out gracefully|Supported|Supported,with sync mode to avoid message loss|Supported|Supported,property filter expressions based on SQL92|Supported|High performance and low latency file storage|Supported timestamp and offset two indicates|Not Supported|Supported, Master-Slave model, without another kit|Supported|Work out of box,user only need to pay attention to a few configurations|Supported, rich web and terminal command to expose core metrics|

具体怎么选， 还是要结合消息队列的特点和应用的场景，比如:

* 如果对于消息队列功能和性能都没有很高的要求，只需要一个开箱即用且易于维护的产品，可以选择 RabbitMQ。
* 如果用于在线业务，比如交易系统，选择 RocketMQ，它低延迟且有着金融级的稳定。
* 如果需要处理海量的消息，比如日志收集、监控信息或者前端埋点，或者使用了大数据、流计算相关的开源产品，那么选择 Kafka。

其他情况，就团队用什么就用什么、自己熟悉什么就用什么。

## 总结 {id="summary"}

在这篇文档中，我们简要介绍了消息队列，其主要复杂在应用之间处理消息的接收、存储以及转发。


接着我们聊了消息队列主要的三个作用，分别是异步处理、流量控制以及服务结偶。延伸开去，还有最终一致性、流量削峰等。

但是，消息队列不是音弹，使用消息队列可能会引入一些额外的问题，比如增加系统的复杂度。

最后我们对比了不同消息队列之间的差异，并提供了技术选型的一些建议，具体选择还需结合应用场景和团队经验。