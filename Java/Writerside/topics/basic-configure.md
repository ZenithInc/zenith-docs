# 基本配置原理

这一篇文档通过 `@Configuration` 注解来实践 SpringBoot 的基本配置原理。

### 使用 @Configuration 和 @Bean

我们先来看如下这段代码，通过`@Configuration` 注解和 `@Bean` 注解也可以将一个类加入到容器中去，实现依赖注入。如下示例:

先创建一个 BannerService 类:

```java
public class BannerService implements IBanner {

    public String getList() {
        return "You are my sunshine!";
    }
}
```

然后创建一个 BannerConfiguration 类:

```java
@Configuration
public class BannerConfiguration {

    @Bean
    public BannerService banner() {
        return new BannerService();
    }
}
```

最后在 `BannerController` 类中注入应用:

```java
public class BannerController {

    private final IBanner service;

    public BannerController(IBanner banner) {
        this.service = banner;
    }

    @GetMapping("/test")
    public String test() {
        return this.service.getList();
    }
}
```

那么这样的一种形式，和之前使用`@Service` 或者 `@Component` 注解有什么区别呢？看上去反而更加复杂了。

### 基础的作用

上面的这个示例只是 `@Configuration` 最基本的使用，并且通过这个示例我们留下了一个问题，就是它和 `@Component` 以及延伸出的这些模式注解之间有什么区别呢？

最基本的一个区别就在于使用`@Configuration` 注解可以自行实例化我们的类，比如说我们的 BannerService 类中存在一个有参的构造函数，我们就可以通过下面的代码自定义的实例化这个类:

```java
@Configuration
public class BannerConfiguration {

    @Bean
    public BannerService banner() {
        return new BannerService("Title", "URL", "Position");
    }
}
```

### 本质上是为了代替原本 XML 配置

`@Configuration` 注解是在 SpringBoot 3 的版本中引入的，本质上是为了代替原有的 XML 中`Beans` 的配置。我们可以来看一眼原来的配置，大致如下：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/LkzcvP1BaSYRS6byCPj4.png" alt="xml"/>

之所以，以前 Java 中大量的使用 XML 的配置是因为，可以配置的内容十分丰富。那么为什么要通过配置去实现呢？因为修改配置是容易的、不容易出错的，而修改代码则是危险的。通过把变化的内容隔离到配置文件中，可以让程序更加的稳定、可扩展。

然后我们将 XML 配置改成了一个 Configuauration 的 Java 类，那么这种做法是否违反了 OCP 的原则呢？

这要看从那个角度来理解。当你把这个 Java 类当作是一个配置的时候，就没有违反，只是配置的格式从 XML 改成了 Java 而已。

从另一个角度而言，也是违反了的。相对于 XML 的配置文件，每次修改配置类都需要重新编译、重新部署、而且 Java 的相对复杂的语法也干扰了配置，相对于 Json、Yaml、XML 等纯粹的配置要复杂很多。

然而在 SpringBoot 中，通常是通过 Configuration 类和配置文件共同来应对变化的。比如通过 Configuration 类来做 MySQL 对象的实例化，在这个类中通过读取配置文件来获取 MySQL 的相关配置参与实例化。