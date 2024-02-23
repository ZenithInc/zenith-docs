# Java 的日志体系

Java 的日志体系有一些混乱。这主要是由于在过去的发展过程中出现了许多不同的日志工具和标准，导致目前在实际应用中有很多相互竞争的日志解决方案。比如，常见的日志工具有 Log4j、Logback、Java Util Logging ( JUL ) 和 SLF4J 等，每个工具都有自己的优缺点和适用场景。这篇文档试图去理清整个体系，并给出实践的示例。

## 概览 {id="overview"}

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/FC202Nc90Ao6Zytp03ki.png" alt="java logging overview"/>

其中 SLF4J 和 commons-logging 只是日志接口的定义，并不包含实现。

横向的四个绿色的包，logback、jdk-logging、log4j、log4j2 这 4 个包是负责实现日志打印功能的框架。

比如我们要使用 SLF4J 和 logback 作为日志的接口以及实现，则需要添加 logback-classic 作为桥接包。同理，如果要接入 jdk-logging，就需要添加 slf4j-jdk14 这个包。

<note>
推荐使用 SLF4J 作为日志的输出，因为它可以对接不同的实现。
</note>

而 commons-logging, 可以不使用桥接包就对接 jdk-logging 或 log4j。

如果你的项目中已经存在了 log4j 的实现，希望转换为 logback。这时候就需要使用 log4j-over-slf4j 先转换为 SLF4J, 然后通过 logback-classic 桥接包转换为 logback 的实现。
