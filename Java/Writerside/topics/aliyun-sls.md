# 接入阿里云日志服务(SLS)

在日常的工作中，我们会在程序中输出很多日志，便于故障的排查，还原事故现场、数据加工、分析。这些日志最初保存在服务器或者关系型数据库中。但是随着日志数量以及查询场景的增多，这样的做法存在很多局限性。这篇文档介绍了在应用程序中集成阿里云的日志服务（**S**imple **L**og **S**ervice)，

### 安装 SDK {id="install"}

首先我们需要安装阿里云的日志服务的相关依赖，这里使用 Maven 作为依赖管理:
```xml
<!-- https://mvnrepository.com/artifact/com.aliyun.openservices/aliyun-log -->
<dependency>
  <groupId>com.aliyun.openservices</groupId>
  <artifactId>aliyun-log</artifactId>
  <version>0.6.96</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.aliyun.openservices/aliyun-log-producer -->
<dependency>
  <groupId>com.aliyun.openservices</groupId>
  <artifactId>aliyun-log-producer</artifactId>
  <version>0.3.17</version>
</dependency>
```

<note>
出于性能的考虑，我们在生产环境中需要使用 `aliyun-log-producer` 这个依赖，通过生产者的方式提交日志。下文也是基于这种方式。
</note>

## 简单示例 {id="simple-example"}

下面是一个最小的示例，演示了如何快速测试体验 SLS:

```Java
import com.aliyun.openservices.aliyun.log.producer.LogProducer;
import com.aliyun.openservices.aliyun.log.producer.ProducerConfig;
import com.aliyun.openservices.aliyun.log.producer.ProjectConfig;

ProjectConfig projectConfig = new ProjectConfig(project, endpoint, accessKey, secretKey);
LogProducer producer = new LogProducer(new ProducerConfig());
producer.putProjectConfig(projectConfig);
```
首先，创建一个`ProjectConfig` 配置对象，它需要传入几个参数:

| 参数名称        | 描述                          | 示例                                |
|-------------|-----------------------------|-----------------------------------|
| `project`   | 项目的名称，在 SLS 服务后台创建 Logstore |                                   |
| `endpoint`  | 接入端点，可以在 SLS 主页看到           | `ap-southeast-1.log.aliyuncs.com` |
| `accessKey` | 阿里云账号授权的 AK                 |                                   |
| `secretKey` | 阿里云账号授权的 SK                 |                                   |

接着，我们通过传入 `projectConfig` 创建了 `producer` 对象，可以通过这个对象投递日志:
```Java
logItem = new LogItem();
logItem.PushBack("created_at", LocalDateTime.now().toString());
logItem.PushBack("context", text);
logItem.PushBack("requestId", requestId);
logItem.PushBack("level", level.name());
logItem.PushBack("timestamp", String.valueOf(System.currentTimeMillis()));
logProducer.send(logProject, logStore, logItem);  // 省略异常处理
```

## 利用 AOP 打印请求日志 {id="print-request-log"}

在 SpringBoot 中，我们可以利用 AOP（面向切面编程）来打印请求日志。采用这样的做法，有如下好处:

* **代码分离和模块化**: AOP 允许将日志记录与业务逻辑分开，对于请求日志来说这种做法非常合适。
* **集中管理** : 避免了在每一个接口中打印请求参数等通用的数据日志，集中化管理。
* **动态代理**: Spring AOP 基于动态代理，因此在运行时可以灵活地应用或移除切面，而无需更改实际的业务代码。
* **提高了代码的可读性**: 由于日志记录和业务逻辑分离，业务方法的代码变得更加清晰和易于理解。

AOP 编程是一种非常好的实践，可以在诸如日志这样的场景不断加以实践。

### 创建注解 {id="define-annotations"}

我们需要创建一个注解，然后将这个注解添加到需要记录请求日志的控制器方法上，就可以实现在这个请求的前后打印请求的相关信息。

```Java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Loggable {}
```

### 创建切面类 {id="define-aspect-class"}

接着，我们创建一个切面类，它起到了定义切面逻辑、配置切点以及数据绑定的功能。在日志场景下，其作用就是配置了在什么时间执行日志打印的操作:

```Java
@Component
@Aspect
public class LoggingAspect {

    @Before("@annotation(com.qunx.promotion.service.infrastructure.attributes.Loggable)")
    public void logMethodEntry(JoinPoint joinPoint) {
        // 获取调用的方法的签名信息
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        // 获取请求信息
        ServletRequestAttributes requestAttributes = (ServletRequestAttributes) 
            RequestContextHolder.getRequestAttributes();
        // 例如，获取请求头中的 Token
        HttpServletRequest request = requestAttributes.getRequest();
            map.put("token", request.getHeader("authorization"));
        // 打印请求日志
    }

    @AfterReturning(pointcut = "@annotation(com.qunx.promotion.service.infrastructure.attributes.Loggable)", returning = "result")
    public void logMethodExit(JoinPoint joinPoint, Object result) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        // 此处打印请求后的日志
        log.info(JSON.toJSONString(result));
    }
}
```
<warning>
需要注意的是，尽量不要在这个地方打印请求参数。一方面，在 Spring 中，`HttpServletRequest` 是迭代对象，只能迭代一次。虽然可以通过 Wrapper 等做法可以实现在此处打印参数，但是对程序性能有较大影响。另外，不加区分的将请求参数输出到日志并不是安全的做法，会泄露敏感数据。如果需要打印请求数据，按照需求，在控制器方法中打印即可。
</warning>

### 使用注解 {id="using-annotation"}

最后，如前文所说，我们就可以在控制器应用注解了:
```Java
@PostMapping("/test")
@Loggable
public UnifyResponse<PromotionUserDetailBO> getDetail(@RequestBody UniqueKeyRequest params, HttpServletRequest request) {
     // 处理业务并返回响应
    return new UnifyResponse<>(result);
}
```

## 在 SpringBoot 中集成 SLS {id="sls-sprint-boot"}

在 SpringBoot 中，我们可以创建一个 `Configuation` 类:
```Java
@Configuration
public class LogServiceConfiguration {

    // 这里可以通过 @Value 注入配置

    @Bean
    public LogProducer logProducer() {
        ProjectConfig projectConfig = new ProjectConfig(project, endpoint, accessKey, secretKey);
        LogProducer producer = new LogProducer(new ProducerConfig());
        producer.putProjectConfig(projectConfig);
        return producer;
    }
}
```
然后我们可以写一个 `LogService` 的类来封装写日志的操作:
```Java
@Service
@RequiredArgsConstructor
public class LogService {
    private final LogProducer producer;
    
    public void write() {
        LogItem item = new LogItem();
        producer.send(logProject, logStore, item); 
    }
}
```

## 使用ID串联日志 {id="using-id"}

在实际的项目中，可以使用一个 id 串联整个请求中的日志。在 API 场景，可以在拦截器中生成一个 UUID，然后业务中写日志可以都带上这个 ID。这样就可以通过一个 ID 查询整个执行链路中的日志。如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/1tw8XW3Y84LRYaQnUxJ2.png" alt="query log"/>

在消费 MQ 的时候，也可以使用 MessageId 来串联整个消费链路的日志。在微服务中，也是通过 traceId 这样的字段，来关联整个请求链路中的日志，日志中也可以记录执行的时间等性能指标，有利于之后分析。

## 总结 {id="summary"}

最后，我们来总结一下在这篇文档中主要介绍了如何在 SpringBoot 应用中接入阿里云的日志服务（Simple Log Service, SLS），以及如何利用 AOP 打印请求日志。文档的要点如下：

1. **安装 SDK**：文档首先指导如何使用 Maven 安装阿里云日志服务的相关依赖，特别指出在生产环境中需要使用 `aliyun-log-producer`。

2. **简单示例**：提供了一个示例，展示了如何使用 SLS。示例中创建了一个 `ProjectConfig` 配置对象和一个 `producer` 对象，用于投递日志。

3. **利用 AOP 打印请求日志**：介绍了在 SpringBoot 中通过 AOP 来打印请求日志的方法，并阐述了使用 AOP 的好处，如代码分离、集中管理、动态代理和提高代码可读性。

4. **在 SpringBoot 中集成 SLS**：指导了如何在 SpringBoot 中创建一个配置类和日志服务类，以封装和简化日志记录的操作。

5. **使用ID串联日志**：建议在实际项目中使用一个唯一ID（如 UUID 或 MessageId）来串联整个请求或消息处理链路中的日志，便于跟踪和分析。

整体而言，文档提供了详细的步骤和代码示例，旨在帮助开发者更有效地在 SpringBoot 应用中集成和使用阿里云日志服务，并通过 AOP 提高日志管理的效率和安全性。