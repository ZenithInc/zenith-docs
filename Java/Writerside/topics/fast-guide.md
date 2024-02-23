# 快速入门

这篇文档描述了关于 SpringBoot 的一些基础内容，包括版本号、热更新的配置以及路由控制器。另外，还描述了如何使用更高级的手法，自动匹配路由前缀，简化路由的编写。

### Spring Boot 的版本号 {id="versions"}

SpringBoot  的版本号是四段式的，如下说明:

```text
主版本号.次版本号.增量版本号.发布说明
```

每一段的说明如下:

- 主版本号： 大规模的变动，一般不向下兼容
- 次版本号：少量的变动，一般要保证向下兼容
- 增量版本号：一般是新增了小的特性或者修复了小的 Bug
- 发布说明：有如下这些说明
    - RC(Release Candidate): 表示是候选版本，不是最终版本
    - Alpha: 表示是内部测试版本，用于给开发和测试用来寻找 Bug 的，不建议一般用户下载使用
    - Bate: 外部测试版本，一般接近于稳定，可能存在少量的 Bug
    - GA: 表示正式发布版本，推荐用户下载使用的版本

### Spring Boot 配置热更新 {id="hot_updated"}

首先，要在`pom.xml` 配置文件中加入依赖的开发者工具包:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <scope>runtime</scope>
  <optional>true</optional>
</dependency>
```

然后，需要配置 IntelliJ IDEA 中配置两处地方。勾选自动构建项目，当我们修改了代码之后就会重新编译代码，生成目标文件:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/CWTBWoxnQDXNvOw4eLEJ.png" alt="configure_1" />

接着需要在高级设置中配置允许在运行或者调试中自动构建项目:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/COV82r192LchZtJ2Gjq4.png" alt="configure_2" />

然后就可以了，低版本的 IDE 中配置并不在高级设置中，请自行百度。

### 项目配置文件

默认情况下，SpringBoot 的配置是`src/main/resources/application.properties` 文件。比如说我要修改服务的监听端口，如下:

```
server.port=8081
```

但是这种配置文件的格式比较原始，而且写起来比较冗余。 SpringBoot 也支持使用 `yml` 格式的配置文件，相同的配置内容，创建文件`application.yml`，突出了配置项的层级:

```yaml
server:
	port: 8081
```

顺便说一下，这种配置对编写文档也很友好，对于部分 MD 编辑器来说可以语法高亮。

### 多环境配置

在正式的项目中，我们会面临很多的环境，比如开发环境、测试环境、预发环境、正式环境。项目的配置在不同的环境中也会面临不同，SpringBoot 也对此提供了支持。

首先，不管什么环境的配置文件都会加载 `applicaiton` 这个主配置文件。多环境配置文件可以在 `resources` 目录下任意位置，其中开发环境的配置文件的名称是固定的，命名为 `applicaiton-dev`。其他环境可以自己定义，以 `application-` 作为前缀即可，比如 `application-test` 、`application-prod`之类的。

> 注意：不会因为环境变化的配置卸载 `application` 中，而因环境而异的配置写在各自的配置文件中。


采用哪个环境配置呢？在 IDE  开发环境下，默认会使用 `dev`。也可以更改`application` 配置如下：

```yaml
spring:
  profiles:
    active: dev
```

其他部署在服务器环境下，可以通过输出 JAR 包，通过命令行的方式指定：

```shell
# 输出 JAR 包
$ mvn clean package
# 运行 JAR 包
java -jar xxx.jar --spring.profiles.active=prod
```

### 项目的层次结构

接下来我们来关注项目的层次结构，从面向接口编程和目录结构两方面来叙述。

#### 层次之间是否需要面向接口编程

从 Java 面向对象规范来说，是需要使用 `interface**` 去约束的。比如写了一个 `BannerController` ，在当中使用应该的是 `BannerService` ,然后实际实现的是 `BannerServiceImpl` 类。这是为了符合 O  CP 原则。

**但是，在实际的项目中，大部分可以不用这么去写。**因为对于 Service 获取其他层来说，改变是非常少的。而面向接口编程，实际上也是面向抽象编程，也是面向变化编程。为了能够应对变化，就需要在写法上更多的约束，这也形成了负担。比如在大型的项目中，我可能有几百个 Service 文件，那么总共要编写的文件数量就是两倍。而且每次变化的时候，都需要改动更多文件。**但是，并不是说所有的情况都不需要，当能够预料到变化就需要去写，比如说使用策略模式的时候。**

之所以，会出现这种规范和实际不一致的情况。是因为我们平时写的这些类的粒度都非常的大，粒度大就意味着替换的难度非常大。当替换的难度远远高于去修改原有的类的时候，我们就自然不会采用接口这样的方式去约束。所以在实际项目中，我们按需符合规范，而不是为了规范而规范。

#### 目录结构的设计

接下来是目录结构的设计，在 `src/main/java/company` 下创建如下的目录结构。其中的目录分为两种，一种是类的定义，一种是层次。

| 目录类型 | 目录名称              | 作用描述               |
| -------- | --------------------- | ---------------------- |
| 层次     | api.v1/controller/api | 用来存放控制器文件     |
|          | service               | 用来存放逻辑类文件     |
|          | repository            | 用来存放数据类文件     |
| 类定义   | dto                   | 用来存放数据传输类     |
|          | model                 | 用来存放数据实体类     |
|          | validator             | 用来存放数据校验规则类 |
|          | exception             | 用来存放异常对类       |
|          | core                  | 用来存放对框架的增强类 |


### Spring Boot 的路由与控制器

因为 SpringBoot 大量的通过注解来实现 Aop 编程，所以我们可以通过注解来标识那些 Java 类属于控制器。

创建`icu.hacking.missyou.v1.api` 这个包，并在这个包下创建一个名为`BannerController` 的类，编写内容如下:

```java
package icu.hacking.missyou.v1.api;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class BannerController {

    @GetMapping("/test")
    public String test() {
        return "You are my sunshine!";
    }
}

```

重要代码的说明如下:

- `@RestController` 这个注解表示这是一个遵循 Restful 接口风格的控制器类。你有可以通过`@Controller`来表示这个控制器。另外，`@RestController`注解除了包含`@Controller`注解外，还包含了`@ResponseBody`注解，这个注解可以简化 Response 的返回。

- `@GetMapping`注解，表示这个接口只能通过 `GET` 请求方法访问。类似的，你可以可以通过`@PostMapping`或者`@PutMapping` 之类的注解来使用其他 HTTP 请求方法。另外你可以通过 `@RequestMapping`来允许所有 HTTP 请求方法或者多个 HTTP 请求方法访问。

### 获取客户端传参

客户端向服务端传递参数主要分为三种形式：

- 第一种是通过 URL Path 中的参数获取，比如说 `/v1/banner/100` 其中 100 假定就是 banner 的 ID
- 第二种是通过 URL 中的查询参数(Query Params) 获取，比如说 `/v1/banner?id=100`
- 第三种是通过请求体中传递的参数

我们先来看看如何获取 URL Path 中的参数:

```java
@GetMapping("/test/{id}")
public String test(@PathVariable Integer id) {
	// 访问 /test/100 输出 id:100
	return "id:" + id;
}
```

第二种是通过 URL 中的查询参数传递，获取方法如下:

```java
@GetMapping("/test")
public String test(@RequestParam Integer id) {
	return "id:" + id;
}
```

第三种传参往往更为复杂，采用 POST 方法传递请求体，比如我们可以采用一个 DTO 对象来接受参数，如下:

```java
public class PersonDTO {
    String name;
    Integer age;
	// ...省略 getter 和 setter 方法
}
```

控制器代码如下:

```java
@PostMapping("/test")
public String test(@RequestBody PersonDTO person) {
	return "name is " + person.getName() + ", " + person.getAge() + " year's old.";
}
```

使用 POSTMAN 测试如下:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/MPhWNFgo4HuCaNZJYBq9.png" alt="postman_test"/>

### Http Client

在 SpringBoot 中也提供了一个 HTTP Client 的实现，我们可以使用它向第三方发起请求。一个简单的 Get 请求, 获取微信公众号的 AccessToken 为例:

```java
@Test
public void testGet() {
	String api = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid={0}&secret={1}";
	String appId = "wx82fXXX";
	String appSecret = "93070ce5a61e10cXXX";
	RestTemplate client = new RestTemplate();
	String url = MessageFormat.format(api, appId, appSecret);
	String response = client.getForObject(url, String.class);
	System.out.println(response);
}
```

如果是其他方法的请求，比如 POST 可以使用 `client.postForObject()` 之类的方法。

### 拦截器

比如在处理用户登录的时候，我们可能会需要做一些用户限流、权限控制的操作。这些操作，我们可以通过 Spring 中的拦截器(Interceptor) 来实现。

拦截器的实现只需要继承 `HandlerInterceptorAdapter` 类即可, 框架会在某些固定的时间点调用当中覆写的方法。我们只需要在这些方法中实现自己的业务逻辑即可:

```java
@Component
public class PermissionInterceptor extends HandlerInterceptorAdapter {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return super.preHandle(request, response, handler);
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        super.afterCompletion(request, response, handler, ex);
    }
}
```

其中`preHandle` 方法是最先执行的，在请求进入 Controller 之前。而 `postHandle` 是在请求渲染视图之前执行。最后是 `afterCompletion` 方法是在请求处理完成之后执行。根据这些时间点，我们执行实现对应的业务逻辑。比如，要实现上文说的限流或者权限控制，在`preHandle`方法中。实现日志记录则在`afterCompletion` 中。

最后，我们还需要通过配置，将这个拦截器注册到框架中去:

```java
@Component
public class InterceptorsConfiguration implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        WebMvcConfigurer.super.addInterceptors(registry);
		// 可以在这里注册更多的拦截器
        registry.addInterceptor(new PermissionInterceptor());
    }
}
```

### 事件驱动编程 {id="event-driven-programing"}

在开发中，为了实现组件的结偶，我们可以采用事件驱动的编程方式。在 SpringBoot 中，提供了事件驱动编程的封装。

以用户注册为例，我们半小时就可以完成功能的实现。可以随着业务的增加，可能会有越来越多的需求在增加在注册功能中，代码越来越臃肿。伪代码如下:
```Java
@Controller
public class UserController {

  @PostMapping("/register")
  public void register() {
    // 实现注册的功能
    // 注册送积分
    // 注册用户统计
    // ....
  }
}
```
这样的场景，我们就可以使用事件驱动的方式重构:
```Java
public class UserController {

  private final ApplicationEventPublisher applicationEventPublisher;
  
  public void register() {
    // 实现注册的功能
    // 触发注册事件
    UserRegisterEvent event = new UserRegisterEvent();
    event.setUserId(1L);
    applicationEventPublisher.publishEvent(event);
  }
}
```

这样重构之后，之后新增的业务我们就可以通过新增 `Listener` 来实现，这样代码就可以职责单一、符合开闭原则:
```Java
@Service
public class UserScore {
  
  @Listener
  public void afterRegister() {
    // 实现注册送积分
  }
}
```

如果注册不强依赖送积分这个业务，我们就可以给 `afterRegister()` 这个方法加上 `@Async` 的注解。这样它可以异步执行，加快 `register` 这个接口的响应、增加吞吐量。