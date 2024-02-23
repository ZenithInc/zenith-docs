# 消息队列的模型 

消息队列并没有统一的标准，曾经的 JMS 或 AMQP 都和现实的实现脱轨很久了。所以，不同的消息队列有不同的概念，不同的实现。这就给我们学习消息队列
带来了一定的困难，相同的名称在不同的消息队列中可能含义不同，相同的含义有不同的名称。

所以，不管是学习哪一款消息队列，我们先搞清楚主流消息队列的一些核心概念和实现模型。这样有利于对消息队列有一个总体的认识。

## 队列模型 {id="topic-and-queue"}

最初的消息队列，就是一个实现了队列结构的系统。比如 ZeroMQ 就是这样一个基于队列的多线程网络库，可以让我们把消息队列的功能集成到系统进程中。

<note>
队列是先进先出（FIFO:Fist-In-First-Out）的线性表。队列只允许在后端（Rear) 插入，在前端 (Front) 删除。
</note>

所以，早期的消息队列就如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/vRNmdke34MdKQVSbMJPE.png" alt="early message queue model"/>


**队列的结构保证了消息的发送、存储是有序的。** 但是如果有多个消费者同时接受同一个队列中的消息的时候，消息的消费就未必是顺序的了。但可以保证
一个消息只有一个消费者消费。 

这时候，我有一个新的需求了，一条消息可以发送给多个消费者了。那么这中结构就不能满足了，除非生产者往多个队列中写入同一条消息，如果你不嫌麻烦的话。

## 发布订阅模型 {id="publisher-subscriber-model"}

发布订阅模型可以解决上面说到的这个问题，就是一条消息可以被多个消费者订阅并消费。其模型示例图如下:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/kscb7k9bg8IMDoiTs4vF.png" alt="publish subscribe model"/>

订阅者先订阅主题，然后队列收到消息后发送给订阅者列表。**这种模型和队列模型的区别就在于一条消息是否可以被多个消费者消费。如果只有一个消费者的情况下，
就退化成为了队列模型。所以发布订阅模型可以兼容队列模型。** 比如 ActiveMQ 就同时兼容两种模型。

现代的消息队列都是基于发布订阅模型的，除了 RabbitMQ，它仍然使用了队列模型。

## RabbitMQ 模型 {id="rabbitMQ-model"}

RabbitMQ 仍然使用的是队列模型，但是为了解决可以让同一个消息发送给不同的消费者的问题，它在生产者和队列之间加入了 Exchange（交换机）的概念。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/L6OkBZGBiW18O1Uu7ru6.png" alt="rabbitmq model"/>

同一份消息如果希望被多个消费者消费，就通过配置 Exchange 将消息发送到多个消息队列，每个消息队列中都有一份完整的消息数据。

## RocketMQ 模型 {id="rocketMQ-model"}

RocketMQ 在生产者、消费者和主题这些概念上发布订阅模型是一致的。但是 RocketMQ 在这个中间加入队列（Queue) 的概念。如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/ekpZ5aL2mUmWICOzL7UY.png" alt="rocketmq model"/>

和之前不一样的是，我们看到 RocketMQ 一个主题中包含了多个队列。为什么要这样设计呢？在解释这个问题之前，我们先来说消息队列的一个机制，
消息确认机制。

当生产者向 MQ 发送一条消息之后，MQ 会返回生产者一个响应，生产者如果收到了响应就表示消息已经发送成功，反之它就应该重发消息。

当 MQ 收到消息后就会向消费者推送消息，消费者收到消息之后，就需要向 MQ 发送一个确认的响应，表示已经正确消费。反之，MQ 要重发消息。

这样一来，在消费者端就会出现一个问题：如果消费者还没有确认消息已经消费成功，那么为了保证有序消费，就不应该消费下一条消息。这样就不能并发消费消息了。

所以，就设计多了多个队列来解决这个问题，在一个消费组中每个消费者都可以订阅一个队列，一个主题下队列的数量，就是消息消费的并发数。这样队列就可以根据
消息数量进行横向扩展了。

<warning>
RocketMQ 只在主题的队列层面保证消息顺序性，在主题层面是不保证消息的有序性的。如果要在主题层面保证消息的有序性，可以使用 Hash 一致性算法，把一类
消息发送到指定的队列中。比如将根据用户ID计算出 Hash，将Hash映射到指定的队列ID上，这样就可以保证相同用户的消息在相同的队列上了。
</warning>

相同消费组内的消费者不会重复消费消息，但是不同消费组可以消费同一条消息。

## Kafka 模型 {id="kafka-model"}

Kafka 的消费模型和 RocketMQ 是一样的，不一样的是 Kafka 中不叫队列，而是叫做分区（Partition）。虽然名称不一样，但是含义和功能是一样的。

> 虽然含义和功能是一样的，但是实现是完全不一样的。

## 总结 {id="summary"}

在学习和使用消息队列时，理解不同消息队列的核心概念和实现模型是至关重要的。所以这篇文档从队列模型说起，接着说了发布订阅模型。现代的消息队列一般都
是基于发布订阅模型来实现的，除了 RabbitMQ。

然后，我们分别介绍了 RabbitMQ、RocketMQ 以及 Kafka 队列的消息模型。