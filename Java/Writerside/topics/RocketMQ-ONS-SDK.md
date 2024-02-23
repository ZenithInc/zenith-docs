# RocketMQ ONS SDK

这篇文档记录了如何通过阿里云官方提供的 RocketMQ ONS SDK 接入阿里云的消息队列服务，并提供了完整的示例程序。

## Install {id="install"}

首先通过 Maven 安装依赖:

```xml
<!-- https://mvnrepository.com/artifact/com.aliyun.openservices/ons-client -->
<dependency>
    <groupId>com.aliyun.openservices</groupId>
    <artifactId>ons-client</artifactId>
    <version>2.0.6.Final</version>
</dependency>
```
<warning>
需要注意：阿里云目前 RocketMQ 实例氛围 4.x 和 5.x 版本，其中4.x 版本某些区域是不支持 2.x 版本的SDK的。
</warning>

## Producer Examples {id="producer"}

生产者接入需要如下的配置信息:

| 配置项                    | 配置描述                      |
|------------------------|---------------------------|
| `AccessKey`            | 阿里云的AK                    |
| `SecretKey`            | 阿里云的SK                    |
| `SendMsgTimeoutMillis` | 发送消息超时时间,例如 "3000" 表示 3 秒 |
| `NAMESRV_ADDR`         | 命名服务的地址                   |

<note>
在发送消息之前，需要首先确认已经在阿里云 Rocket MQ 管理后台创建了 Topic。
</note>

### 发送普通消息 {id="send_simple_message"}

发送普通消息的示例代码如下：

<procedure>
<step>
首先需要配置属性:

```Java
Properties properties = new Properties();
properties.put(PropertyKeyConst.AccessKey, "xxxx");
properties.put(PropertyKeyConst.SecretKey, "xxxx");
properties.put(PropertyKeyConst.SendMsgTimeoutMillis, "3000");
properties.put(PropertyKeyConst.NAMESRV_ADDR, "http://MQ_INST_xxxx.ap-southeast-1.mq.aliyuncs.com:80");
```
</step>
<step>
然后构造并启动生产者:

```Java
Producer producer = ONSFactory.createProducer(properties);
producer.start();
```
</step>

<step>
构造消息对象:

```Java
Message message = new Message("DEV_JUN_XIAO_TEST", "*", "Hello MQ".getBytes());
message.setKey(UUID.randomUUID().toString());
```
</step>

<step>
发送消息，发送完成之后关闭生产者(如果频繁发送，可以不主动关闭，节省内存资源):

```Java
producer.send(message);
producer.shutdown();
```
</step>
</procedure>

### 消费消息 {id="consumer"}

消费消息和生产消息所需的配置基本相同，唯需要指定消费者组（Consumer Group）。消费者组需要提前在阿里云和后台配置。

首先需要创建一个`Configuation` 类，写入配置信息并启动消费线程、设置事件监听者:
```Java
@Configuration
@RequiredArgsConstructor
public class OptionChargedSuccessConsumerConfiguration {

    private final MessageListener listener;
    
    // 可以通过 @Value 注入配置

    /**
     * 获取属性配置
     */
    private Properties getProperties() {
        Properties properties = new Properties();
        properties.setProperty(PropertyKeyConst.NAMESRV_ADDR, nameSrvAddr);
        properties.setProperty(PropertyKeyConst.GROUP_ID, consumerGroup);
        properties.setProperty(PropertyKeyConst.AccessKey, accessKey);
        properties.setProperty(PropertyKeyConst.MessageModel, PropertyValueConst.CLUSTERING);
        properties.setProperty(PropertyKeyConst.SecretKey, secretKey);
        return properties;
    }

    /**
     * 配置构建消费者
     */
    @Bean(initMethod = "start", destroyMethod = "shutdown")
    public ConsumerBean buildConsumer() {
        ConsumerBean consumer = new ConsumerBean();
        Properties properties = getProperties();
        consumer.setProperties(properties);
        Map<Subscription, MessageListener> subscriptions = new HashMap<>();
        Subscription subscription = new Subscription();
        subscription.setTopic(topic);
        subscription.setExpression("*");
        subscriptions.put(subscription, listener);
        consumer.setSubscriptionTable(subscriptions);
        return consumer;
    }
}
```

消费者的代码如下:
```Java
@Service
@RequiredArgsConstructor
public class OptionChargedSuccessListener implements MessageListener {

    private final ApplicationEventPublisher applicationEventPublisher;

    @Override
    @Transactional
    public Action consume(Message message, ConsumeContext consumeContext) {
        // 收到的消息
        String body = new String(message.getBody(), StandardCharsets.UTF_8);
        // 可以通过事件通知消费者 
        applicationEventPublisher.publishEvent(event.get());
        // 省略了异常处理
        return Action.CommitMessage;

    }
}
```
通过事件的方式在此转发消息是我推荐的做法，这样避免了随着业务的增加、消费者的增加，在 `MessageListener` 中的代码越来越多。也可以统一做异常的处理、日志的记录。

## 总结 {id="summary"}

这篇文档介绍了接入阿里云的 RocketMQ 的 SDK，通过 SDK 生产以及消费消息，并给出了完整的示例代码。