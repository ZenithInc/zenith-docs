# 关于 Java

这篇文档描述了 Java 的一些背景信息，比如名字的由来。

## Java 名字的由来 {id="name"}

Java 这个名字本意是印度尼西亚爪洼岛的英文，因为盛产咖啡闻名，Java 之父 James Gosling 一开始取了两个名字 Lyric 和 Oak, Oak 是他办公室外一
颗茂密的橡树。但是这两个名字都已经被注册了，所以大家又想了很多名字，并且投票投出了 Silk，中文名为丝绸。但是 James Gosling 反对。然后排在第二
和第三的这两个名字，律师反对。于是就选择了排在第四的名字，叫做 Java。

另外，一个 Java 的 Class 编译后的字节码文件如果用 16 进制来查看的话，开头是`CAFE BABA`, 所以 Java 和咖啡有着非常紧密的联系:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/smOI6LslYYdjNF7quqto.png" alt="bytes code"/>

## 字节码名字的由来 {id="bytecode"}

因为 JVM 中一个指令中第一个字节代表操作码（opcode），所以被称之为字节码。一些常见的字节码，如下表所示:

| 指令类别    | 示例指令                                                               | 描述                |
|---------|--------------------------------------------------------------------|-------------------|
| 常量加载    | iconst_m1, iconst_0, ldc (加载常量池中的常量)                               | 将整型或引用常量压入操作数栈    |
| 数组和集合操作 | newarray (创建新的基本类型数组), aastore (将引用类型值存入数组)                        | 创建、读取或修改数组元素      |
| 控制流     | goto, ifeq (如果等于零则跳转), jsr (跳转并保存返回地址)                             | 改变程序执行顺序          |
| 方法调用与返回 | invokevirtual, invokestatic, invokespecial, invokevirtualinterface | 调用类的方法或接口方法以及处理返回 |
| 类型转换    | i2l (int转long), checkcast (检查对象是否为指定类或其子类实例)                       | 在运行时进行数据类型的转换     |
| 局部变量操作  | iload_n, istore_n (加载/存储int型局部变量)                                  | 从局部变量表中读取或写入数据    |
| 数学运算    | iadd, isub, imul, idiv (整数加减乘除)                                    | 对整数执行数学运算         |
| 对象操作    | anewarray (创建新的引用类型数组), getfield, putfield                         | 创建新对象、访问或修改对象字段   |

## OpenJDK 和 OracleJDK {id="difference-between-openjdk-and-oracle-jdk"}

简单来说，OpenJDK 和 OracleJDK 它们之间的差异非常小。一般情况下，我们选择 OpenJDK 就可以了，有着更宽容的许可。

| 特性        | OpenJDK                                                                | Oracle JDK                                                |
|-----------|------------------------------------------------------------------------|-----------------------------------------------------------|
| **许可证**   | GNU General Public License, version 2 (GPLv2) with Classpath Exception | Oracle Technology Network License Agreement（对于商业用途可能需要付费） |
| **支持**    | 社区支持，部分第三方提供商提供商业支持                                                    | Oracle提供的专业商业支持                                           |
| **性能和特性** | 作为Java平台的参考实现，与Oracle JDK非常接近                                          | 过去包含一些独有特性和性能优化，但现在两者差异较小                                 |
| **更新和维护** | 定期更新，但商业支持可能不如Oracle JDK                                               | 对于商业用户，提供优先级较高的更新和安全补丁                                    |
| **适用场景**  | 适用于需要开源解决方案的开发者和企业，以及大多数通用应用                                           | 适合需要额外商业支持和服务的企业用户                                        |


## 让人困惑的版本号 {id="version"}

针对不同的目标平台，Java 有如下版本:

| Java版本                       | 目标平台     | 主要用途                                                                                                    |
|------------------------------|----------|---------------------------------------------------------------------------------------------------------|
| Java SE (Standard Edition)   | 个人电脑、服务器 | Java SE提供了进行Java应用程序开发所需的核心功能，包括所有的基本APIs和Java虚拟机（JVM）。它是开发桌面应用程序、控制台程序和基本服务器应用程序的基础。                   |
| Java EE (Enterprise Edition) | 企业级服务器   | Java EE在Java SE的基础上增加了构建大型、多层、可扩展、安全且高性能的企业级应用程序的功能。它包括一系列扩展APIs和运行时环境，用于开发面向企业的应用程序，比如Web服务、分布式计算和云应用。 |
| Java ME (Micro Edition)      | 嵌入式和移动设备 | Java ME是为嵌入式和移动设备提供的轻量级平台，支持有限资源的设备，比如手机、掌上游戏机和小型嵌入式设备。它包括一套特定的APIs，用于开发适合这些类型设备的应用程序。                  |

随着技术的发展，Java ME 的重要性已经大大降低，因为智能手机和平板电脑等设备已经足够强大，能够运行标准的Java SE应用程序。同时，Java EE已经被 
Jakarta EE所取代，这是由 Eclipse Foundation 管理的开源项目，继续 Java EE 的工作和进步。Jakarta EE 旨在发展云原生Java企业级应用的新技术
和标准。

Java SE 传统的版本表示法是 1.x。比如 Java 8 也可以表示为 Java 1.8。但是从 Java 9 开始就不再如此表示了。

| Java版本     | 发布日期    | 字节码版本 | 主要特性                                                                |
|------------|---------|-------|---------------------------------------------------------------------|
| Java SE 8  | 2014年3月 | 52    | Lambda表达式、Stream API、新的日期时间API、接口的默认和静态方法                           |
| Java SE 9  | 2017年9月 | 53    | 模块系统（Jigsaw项目）、JShell、私有接口方法                                        |
| Java SE 10 | 2018年3月 | 54    | 局部变量类型推断（var关键字）、应用类数据共享（CDS）                                       |
| Java SE 11 | 2018年9月 | 55    | 新的HTTP客户端API（标准化）、局部变量语法扩展（lambda表达式）、ZGC                           |
| Java SE 12 | 2019年3月 | 56    | Shenandoah GC、Switch表达式（预览）、JVM常量API                                |
| Java SE 13 | 2019年9月 | 57    | Switch表达式（标准）、文本块（预览）、ZGC和Shenandoah GC的改进                          |
| Java SE 14 | 2020年3月 | 58    | Switch表达式（标准化）、记录（Record）（预览）、Pattern Matching for instanceof（预览）   |
| Java SE 15 | 2020年9月 | 59    | 文本块（标准）、Sealed类（预览）、ZGC和Shenandoah GC的改进、隐藏类                        |
| Java SE 16 | 2021年3月 | 60    | 记录（Record）（标准）、Pattern Matching for instanceof（标准）、ZGC的改进           |
| Java SE 17 | 2021年9月 | 61    | 封密类（Sealed Classes）（标准）、Pattern Matching for switch（预览）、新的垃圾回收器JEPs |
| Java SE 18 | 2022年3月 | 62    | 项目Loom的结构化并发（预览）、Project Panama的向量API（孵化器）                          |
| Java SE 19 | 2022年9月 | 63    | 结构化并发（孵化器）、记录模式（预览）、外部函数和内存API（预览）、虚拟线程（预览）、Switch 表达式（预览）          |
| Java SE 20 | 2023年3月 | 64    | 作用域值(孵化器）、向量API（孵化器）、结构化并发（孵化器）                                     |
| Java SE 21 | 2022年9月 | 65    | 虚拟线程、有序集合、外部函数和内存API(预览)、作用域值（预览）、Vector API(孵化器）、Record 模式、分代 ZGC  |
