# 条件注解

我们可以使用模式注解，注入一个接口中的某个实现类。如果一个接口有多个实现类，我们可以通过好几种方式来选择其中的一个类。比如通过 byname 的方式，或者通过`@Qualifier` 注解来指定一个类，或者通过注释某个类上的 `@Component` 注解来实现，或者通过 `@Primary` 注解来优先指定某个类。

这篇文档中，解释另外一种更好的方式，叫做条件注解。

### 条件注解的基本使用

我们仍旧以之前的 Banner 为例。首先创建一个 BannerService 类:

```java
public class BannerService implements IBanner {
    public String getList() {
        return "You are my sunshine!";
    }
}
```

然后创建一个 MockBannerService 类:

```java
public class MockBannerService implements IBanner{
    @Override
    public String getList() {
        return "Mock: You are my sunshine!";
    }
}
```

这两个类都实现了 IBanner 接口，这个接口中就一个 `getList` 方法，实行实现即可。然后我们再实现一个名为 `BannerController` 的控制器类，用来注入这两个类:

```java
@RestController
public class BannerController {

    private final IBanner service;

    public BannerController(IBanner iBanner) {
        this.service = iBanner;
    }

    @GetMapping("/test")
    public String test() {
        return this.service.getList();
    }
}
```

然后写一个 Configuration 类，来应用条件注解:

```java
@Configuration
public class BannerConfiguration {

    @Bean
    @Conditional(BannerCondition.class)
    public BannerService banner() {
        return new BannerService();
    }

    @Bean
    @Conditional(MockBannerCondition.class)
    public MockBannerService mock() {
        return new MockBannerService();
    }
}
```

接着，我们来实现上面的两个条件注解类:

```java
public class BannerCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return true;
    }
}
public class MockBannerCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return false;
    }
}
```

这样我们就完成了所有的代码编写，可以通过上面代码改变第4行和第10行中的 `true` 和 `false` 来控制具体那个类加入到容器中。

### 上下文的传参

我们要在 `matches` 方法中进行条件判断，就需要能够获取上下文中的一些信息。而向上文中那样，纯粹返回一个布尔值是没有任何意义的。如下示例中，我们通过获取配置文件中的字段进行判断:

```java
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    String value = context.getEnvironment().getProperty("key");
    return Objects.requireNonNull(value).equalsIgnoreCase("okay");
}
```

如果只是从环境变量中读取对应的配置进行判断的话，我们可以使用另外一个条件注解来简化编写, 如下所示:

```java
@Configuration
public class BannerConfiguration {
    @Bean
//    @Conditional(BannerCondition.class)
    @ConditionalOnProperty(value = "okay", matchIfMissing = true)
    public BannerService banner() {
        return new BannerService();
    }
}
```

参数`value` 表示要进行对比的值，而`matchIfMissing` 表示如果配置不存在则是否注入。除了`@ConditionalOnProperty` 这个外，还有很多 Spring 已经封装好的条件注解，如下不完全所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/XFRplWMZmzg9s1KgOQ8f.png" alt="source"/>