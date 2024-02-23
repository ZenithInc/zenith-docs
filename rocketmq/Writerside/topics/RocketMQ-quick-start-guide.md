# RocketMQ 快速入门

这篇文档介绍了 RocketMQ 的基础使用，包括生产者以及消费者的 API。通过阅读本文档，可以帮助你快速掌握并应用 RocketMQ。

> 需要注意的是，本文的环境使用的是阿里云 RocketMQ v5.x 版本，SDK 使用的是社区版本。示例代码使用 Java 编写。

## 安装 SDK {id="install-sdk"}

首先在 Maven 的 `pom.xml` 文件中，指定 SDK 以及版本:
```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client-java</artifactId>
    <version>5.0.4</version>
</dependency>
```

## 生产者 API {id="producer-api"}

接着来我们介绍生产者的 API，发送各类消息。

<code-block lang="plantuml">
@startuml
left to right direction
package Client {
    usecase "Producer1" as p1
    usecase "Producer2" as p2
}
package Server {
    usecase "Topic1" as t1
    usecase "Topic2" as t2
}
p1 --> t2
p2 --> t1
p1 --> t1
p2 --> t2
@enduml
</code-block>

### 发送同步消息 {id="send-sync-message"}

发送同步消息，需要如下步骤:

<procedure>

<step title="构造配置">

首先，我们需要构造配置对象。因为我是通过公网接入阿里云的 RocketMQ 实例的，所以需要多配置`StaticSessionCredentialsProvider`， 需要从阿里云 MQ 实例详情中获取 `accessKey` 和  `secretKey` 配置。

<code-block lang="java">
StaticSessionCredentialsProvider credentialsProvider = new StaticSessionCredentialsProvider(accessKey, secretKey);
 ClientConfigurationBuilder builder = ClientConfiguration.newBuilder()
     .setEndpoints(endpoint)
    .setCredentialProvider(credentialsProvider);
ClientConfiguration configuration = builder.build();
</code-block>

</step>

<step>
接着我们需要构建生产者（Producer），代码如下:

<code-block lang="java">
ClientServiceProvider provider = ClientServiceProvider.loadService();
Producer producer = provider.newProducerBuilder()
    .setTopics("trades")
    .setClientConfiguration(configuration)
    .build();
</code-block>

</step>

<step>

再接着，我们需要构建简单的 `Hello World` 的消息:

<code-block lang="java">
Message message = provider.newMessageBuilder().setTopic("trades")
    .setKeys(UUID.randomUUID().toString())
    .setTag("*")
    .setBody("Hello World".getBytes())
    .build();
</code-block>

</step>

<step>

最后一步，自然是发送消息了:

<code-block lang="java">
SendReceipt receipt = producer.send(message);
// 输出 MessageId: 0102DFB8838E67E29005542BB800000000
System.out.println(receipt.getMessageId());
producer.close();   // 关闭生产者
</code-block>

</step>
</procedure>

### 发送异步消息 {id="send-async-message"}

如果是发送异步消息，前面步骤和发送同步消息是一致的。 构造配置对象、构造生产者、构造消息。发送消息的代码如下：

```Java
CompletableFuture<SendReceipt> future = producer.sendAsync(message);
ExecutorService sendCallbackExecutor = Executors.newCachedThreadPool();
future.whenCompleteAsync(((sendReceipt, throwable) -> {
    if (null != throwable) { return;} // 异常处理
    System.out.println(sendReceipt.getMessageId());
}), sendCallbackExecutor);
```
首先，它使用 `producer.sendAsync(message)` 方法发送一条异步消息。这将在消息发送完成后返回一个 `CompletableFuture` 对象，该对象代表了一种未来的值，即消息发送成功的回执。

接下来，它创建了一个新的线程池 `sendCallbackExecutor` 来执行发送回调。这是因为异步消息发送完成后，RocketMQ 将会在线程池中执行回调函数，而不是在发送线程中。

最后，它使用 `future.whenCompleteAsync()` 方法注册一个回调函数，当消息发送成功或者出现异常时，该函数将会被调用。在回调函数中，如果发生异常，则进行异常处理；否则，打印出消息 ID。

### 发送定时消息 {id="send-delay-message"}

比如说订单超时的场景，在下单之后可以向队列发送一条定时的消息，比如十五分钟。然后 Broker 会在时间到了之后，给消费者推送这条消息。

构造配置对象、构造生产者、发送消息和上文中的示例都是一样的。区别在于构造消息的时候，需要指定延时参数:
```Java
Message message = provider.newMessageBuilder().setTopic("trades")
    .setKeys(UUID.randomUUID().toString())
    .setTag("*")
    .setBody("Hello World".getBytes())
    // 单位为毫秒
    .setDeliveryTimestamp(System.currentTimeMillis() + 10000L) 
    .build();
```

定时消息的使用需要注意几点：

1. 创建 Topic 的时候，其 MessageType 需要选择“Delay”。 
2. 默认的消息精度为 1000 毫秒(1秒)。
3. 不要投递同一时刻大量的消息，因为消费端需要消费大量消息，会造成延时。

## 消费者 API {id="consumer-api"}

RocketMQ 支持三种消费者，分别是 PushConsumer、SimpleConsumer 以及 PullConsumer。这三种的差别如下:

| 对比项     | Push Consumer                        | Simple Consumer            | Pull Consumer             |
|---------|--------------------------------------|----------------------------|---------------------------|
| 接口方式    | 使用监听器回调接口返回消费结果，消费者仅允许在监听器范围内处理消费逻辑。 | 业务方自行实现消息处理，并主动调用接口返回消费结果。 | 业务方自行按队列拉取消息，并可选择性地提交消费结果 |
| 消费并发度管理 | 由SDK管理消费并发度。                         | 由业务方消费逻辑自行管理消费线程。          | 由业务方消费逻辑自行管理消费线程。         |
| 负载均衡粒度  | 5.0 SDK是消息粒度，更均衡，早期版本是队列维度           | 消息粒度，更均衡                   | 队列粒度，吞吐攒批性能更好，但容易不均衡      |
| 适用场景    | 适用于无自定义流程的业务消息开发场景                   | 适用于需要高度自定义业务流程的业务开发场景。     | 仅推荐在流处理框架场景下集成使用          |
| 接口灵活度   | 高度封装，不够灵活。                           | 原子接口，可灵活自定义。               | 原子接口，可灵活自定义。              |

<warning>
生产环境中相同的 ConsumerGroup 下严禁混用 PullConsumer 和其他两种消费者，否则会导致消息消费异常。
</warning>

### Push Consumer {id="push-consumer"}

Push Consumer 是一个高度封装的消费者，可以通过设置一个 `Listener` 来监听并消费消息。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/BPhoqhO1f9xMJB80KozF.png" alt="rocketmq push consumer"/>

完成一条消息消费的过程如下:

<procedure>
<step>
构造配置，这个和上文中的 Producer 是一样的。

<code-block lang="java">
StaticSessionCredentialsProvider credentialsProvider = 
    new StaticSessionCredentialsProvider(accessKey, secretKey);
ClientConfigurationBuilder builder = ClientConfiguration.newBuilder().setEndpoints(endpoint)
   .setCredentialProvider(credentialsProvider);
ClientConfiguration configuration = builder.build();
</code-block>

</step>

<step>
接着，我们需要构造一个 Push Consumer 对象:

<code-block lang="java">
ClientServiceProvider provider = ClientServiceProvider.loadService();
// 消息过滤，后文再详细说
FilterExpression filter = new FilterExpression("*", FilterExpressionType.TAG);
provider.newPushConsumerBuilder()
        .setSubscriptionExpressions(Collections.singletonMap("trades", filter))
        .setClientConfiguration(getConfiguration())
        .setConsumerGroup("trades");
</code-block>

</step>

<step>
最后，设置一个 Listener 来监听并消费消息，然后返回 `ConsumerResult.SUCCESS`:

<code-block lang="java">
provider.setMessageListener(messageView -> {
    messageView.getBody().flip();
    String message = StandardCharsets.UTF_8.decode(messageView.getBody()).toString();
    System.out.println("message = " + message);
    return ConsumeResult.SUCCESS;
}).build();
</code-block>

</step>

</procedure>

<warning>
Push Consumer 需要严格控制消息的消费时间，尽量避免出现消息超时导致重复消费。且必须按照同步的方式进行消费，在消费真正成功后在返回 `ConsumerResult.SUCCESS` 。
</warning>

## 接入 SpringBoot {id="spring-boot"}

接着，我们可以尝试在 SpringBoot 中接入 RocketMQ。这里使用 Maven 进行演示，首先申明依赖:
```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>${rocketmq.spring.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>${rocketmq.version}</version>
</dependency>
```

然后在项目的配置文件中加入 RocketMQ 相关的配置:
```yaml
rocketmq:
  name-server: 47.236.128.181:9876
```

接着，我们就可以直接写一个消费者的 `Listener` 了，代码也非常简洁:
```Java
@Service
@RocketMQMessageListener(topic = "TEST_TRADE_SUCCESS", consumerGroup = "promotion")
public class TradeSuccessConsumer implements RocketMQListener<MessageExt> {

    @Override
    public void onMessage(MessageExt messageExt) {
        // todo: 编写消费的逻辑
    }
}
```
如有需要，可以把 `topic` 定义在配置文件中，这样的话在 `@RocketMQMessageListener` 注解中可以使用 SpEL 表达式，例如: `${rocketmq.topic}`。

如有需要，可以在 `@RocketMQMessageListener` 注解中添加消费线程相关的配置，例如 `consumeThreadMin` 和 `consumeThreadMax`。