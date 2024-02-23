# Spring IOC 的核心机制:实例化与注入

在 SpringBoot 中提供了下面这些模式注解。其中最基础的便是`@Component`注解。

由`@Component`这个注解衍生出来下面这几个注解，只有语义上的区别，没有本质上的区别:`@Service`、`@Controller`、`@Repository`。

最后是`@Configuration`注解，这个注解更加灵活，允许更多的自定义。

### Spring 中的依赖注入的方式

可以通过属性注入的方式。首先创建一个 Service 的类，代码如下:

```java
package icu.hacking.missyou.v1.service;

import org.springframework.stereotype.Service;

@Service
public class BannerService {

    public String getList() {
        return "You are my sunshine!";
    }
}
```

`@Sergvice`这个注解会将这个类加入到容器中去。然后依赖这个类的属性上，采用`@autowired`注解来注入，容器会自动实例化并注入。

```java
@RestController
public class BannerController {
    @autowired
    private final BannerService service;

    @GetMapping("/test")
    public String test() {
        return this.service.getList();
    }
}
```

但是并不建议采用这种属性注入的方式，因为这种方式破坏了面向对象的封装特性。你可以看到 `service` 属性的可见性是`private`，不应该从外部访问或是注入。所以更推荐采用构造器注入的方式，代码更改如下:

```java
@RestController
public class BannerController {

    private final BannerService service;

    public BannerController(BannerService service) {
        this.service = service;
    }

    @GetMapping("/test")
    public String test() {
        return this.service.getList();
    }
}
```

此外，也可以通过 `setter` 方法注入，更改代码如下:

```java
@RestController
public class BannerController {

    private BannerService service;

    @Autowired
    public void setService(BannerService service) {
        this.service = service;
    }

    @GetMapping("/test")
    public String test() {
        return this.service.getList();
    }
}
```

### 模式注解的启动时机

在 SpringBoot 中，采用了模式注解的类是在什么时候被实例化的呢？是在应用启动的时候，还是在访问这个类的时候呢？默认情况下是在应用启动的时候，我们可以通过如下实验来证明:

```java
@Service
public class BannerService {

    public BannerService() {
        System.out.println("Service Init");
    }

    public String getList() {
        return "You are my sunshine!";
    }
}
```

在这个类中，我们加入了一个无参的构造函数，其中输出一行字符。然后我们重新启动 SpringBoot 的应用，出现如下输入:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/069Y8hjv7DD5SYvFKmVM.png" alt="output"/>

由此可见，应用启动的时候就已经实例化了我们的类。那么可不可以延迟启动呢？可以，在这个类上加入`@Lazy` 的注解。我们来试试看:

```java
@Service
@Lazy
public class BannerService {// ...省略...}
```

实验的结果表明，并没有任何作用，这个类仍然会在应用启动的时候被实例化。这是为什么呢？因为我在 Controller 类中已经依赖了这个类，而 Controller 并没有延迟启动，所以在 Controller 类被启动的时候，其依赖的其他的类也会被启动不管是否标记了`@Lazy`。

所以解决方法就是在 Controller 类上，也标记为`@Lazy`:

```java
@RestController
@Lazy
public class BannerController {//...省略...}
```

当我们第一次去访问 Controler 类的时候，依赖的 Service 类会被实例化。

### 注入接口的实现类

前面我们注入的都是具体的类，但是从面向对象的角度来说，这样的注入是没有意义的，因为具体的类就写死了不能有效应对变化。那么我们如何应对变化呢？就是采用面向接口的编程方式。所以，我们注入的不应该是具体的类，而是抽象的接口。

如果注入的是接口，那么有两种注入方式，一种是`ByName`一种是`ByType`。我们先来演示 `ByType` 的方式。

新建一个类名为`IBanner`，代码如下:

```java
public interface IBanner {
    public String getList();
}
```

然后新建一个类名为`BannerService`的实现类:

```java
@Service
public class BannerService implements IBanner {
    public String getList() {
        return "You are my sunshine!";
    }
}
```

最后，在 Controller 类中注入应用:

```java
@RestController
public class BannerController {

    private final IBanner service;

    public BannerController(IBanner service) {
        this.service = service;
    }

    @GetMapping("/test")
    public String test() {
        return this.service.getList();
    }
}
```

这个时候，最后会成功输出`You are my sunshine!` 的字样，说明注入了`BannerService` 这个实现类。这说明，SpringBoot 会自动推断出容器中实现了 `IBanner` 接口的类并注入。那么如果容器中有多个实现了该接口的类呢？我们再来写一个实现类:

```java
@Service
public class MockBannerService implements IBanner{
    @Override
    public String getList() {
        return "Mock: You are my sunshine!";
    }
}
```

在有多个实现类的情况下，SpringBoot 会选择注入哪一个呢？首先会根据注入的变量名为推断，比如说下面的构造函数中的传参改一下，注入的就是`MockBannerService` 类:

```java
public BannerController(IBanner mockBannerService) {
    this.service = mockBannerService;
}
```

这种就是根据`ByName`的方式注入。