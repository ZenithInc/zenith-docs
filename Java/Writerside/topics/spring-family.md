# Spring Framework

这篇文档描述了 Spring Framework 的相关内容。早期阶段，它指的是实现了 MVC,IOC 的 Java 企业级框架，现在指的是 spring.io 下的所有的项目，包括 SpringBoot、Spring MVC、Spring Data、Spring Cloud 等。

### 发展历史

Spring 的发展历史如下表不完全展示:

<code-block lang="mermaid">
timeline
    Title History of The Spring Framework
    2002.10 : Rod Johnson 在 Expert One-on-One J2EE 设计和开发的书中提供了 Spring 最初的源码
    2003.02 : Rod Juergen 和 Yann 创建了开源项目 Spring
    2004.03 : Spring1.0 发布 : Interface21 公司快速发展
    2007.11 : Spring2.5 发布 : 支持注解配置 : 更名为 SpringSource
    2009.12 : Spring3.0 发布 : 支持基于 Java 的 Bean 注解
    2013.12 : Spring4.0 发布 : 公司为 Pivotal 支持 Java8
    2014.04 : SpringBoot 1.0 发布
    2014.10 : Spring Cloud 1.0 发布
    2017.09 : Spring 5.0 发布 : 支持响应式编程模型
    2018.03 : SpringBoot 2.0 发布 : 基于 Spring 5 构建
    2022.11 : SpringBoot 3.0 发布 : 基于 Java17 : 支持 Java 19 G : GraaLVM 支持
    
</code-block>

### 生态环境

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/EjsTa2Xu99FCOTQ0f2HJ.png" alt="family"/>

### 设计哲学

Spring 的设计哲学来自[官方网站](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html)的描述，一共有 5 条，中英文翻译如下:

> Provide choice at every level. Spring lets you defer design decisions as late as possible. For example, you can switch persistence providers through configuration without changing your code. The same is true for many other infrastructure concerns and integration with third-party APIs.

提供各个层次的选择。Spring 允许您尽可能推迟设计决策。例如，可以通过配置切换持久性提供程序，而无需更改代码。对于许多其他基础设施问题和与第三方 API 的集成也是如此。


> Accommodate diverse perspectives. Spring embraces flexibility and is not opinionated about how things should be done. It supports a wide range of application needs with different perspectives.

包容不同的观点。Spring 支持灵活性，对于事情应该如何处理并不固执己见。它以不同的视角支持广泛的应用程序需求。


> Maintain strong backward compatibility. Spring’s evolution has been carefully managed to force few breaking changes between versions. Spring supports a carefully chosen range of JDK versions and third-party libraries to facilitate maintenance of applications and libraries that depend on Spring.

保持向下兼容。Spring 的演变已经被精心管理，以便在不同版本之间几乎不发生重大变化。Spring 支持一系列精心挑选的 JDK 版本和第三方库，以方便维护依赖 Spring 的应用程序和库。


> Care about API design. The Spring team puts a lot of thought and time into making APIs that are intuitive and that hold up across many versions and many years.

关注 API 设计。Spring 团队投入了大量的精力和时间来制作直观的 API，这些 API 可以支持多个版本和多年。


> Set high standards for code quality. The Spring Framework puts a strong emphasis on meaningful, current, and accurate javadoc. It is one of very few projects that can claim clean code structure with no circular dependencies between packages.

为代码质量设置高标准。Spring 框架强调有意义、当前和准确的 javadoc。它是为数不多的可以声称没有包之间循环依赖关系的干净代码结构的项目之一。


### 架构特征

支持 Web 应用、无服务架构、事件驱动、批处理、微服务、响应式、云原生。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/D23Zq7i0nK6uHKXMkULX.jpeg" alt="futures"/>

### 核心模块

这部分内容可以参考[官方文档](https://docs.spring.io/spring-framework/docs/4.0.x/spring-framework-reference/html/overview.html):

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/qjrPBrAnHcUk1XD27sa7.png" alt="core"/>