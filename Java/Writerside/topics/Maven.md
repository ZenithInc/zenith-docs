# Maven

Maven 是 Apache 基金会开发的顶级开源项目，为项目工程管理的工具，对软件项目提供构建与依赖管理。Maven 的出现，让 Java 工程的创建进入到了标准化的时代。

<note>
Apache 软件基金会(Apache Software Foundation)， 是专门为支持开源项目而办的一个非盈利性组织。在它所支持的 Apache 项目与子项目中，所发行的软件产品都遵循 Apache 许可证(Apache License)。
</note>

## 核心特性 {id="core_futures"}

Maven 的核心特性如下所示：

* 项目设置遵循统一的规则，保证不同开发环境的兼容性
* 强大的依赖管理，项目依赖组件自动下载，自动更新
* 可扩展的插件机制，使用简单，功能丰富

## 安装和配置 {id="install_configure"}

下载安装使用 Maven 需要满足一些前提条件：

* JDK 版本必须在 Java1.7 以及以后版本才可以
* 磁盘必须大于 10 MB，本地仓库最少 500MB

在 MacOS 上，需要使用如下命令安装:

```Shell
brew install maven
mvn --version
```

## 依赖管理 {id="dep_manager"}

Maven 利用 dependency自动下载并管理第三方的 Jar。从远程的 Apache 的中央仓库进行下载到本地仓库，在工程中进行引用。

```xml
<dependency>
  <groupId>com.belerweb</groupId>
  <artifactId>pinyin4j</artifactId>
  <version>2.5.5</version>
</dependency>
```

解释一下这当中标记对的含义:

* GroupId: 机构或者团体的英文，采用“反转域名”的书写形式
* ArtifactId: 项目名称，说明其用途，例如 cms, oa
* Version： 版本号，一般采用版本 + 单词的书写形式，比如 1.0.0.RELEASE

这些第三方的依赖，我们可以在`https://search.maven.org` 这个网站上搜索，如下截图:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/thwa3WetrMG4IfadLm6b.png" alt="search maven"/>

点击版本号进入详情就可以看到详情了:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/erd2qIr9NdwIMpZ2Tjc8.png" alt="search detail" />

然后将其加入到我们的pom.xml配置文件的 dependencies 标记对中然后让 IDE 去下载同步即可:

```xml
<dependencies>
  <dependency>
    <groupId>com.belerweb</groupId>
    <artifactId>pinyin4j</artifactId>
    <version>2.5.1</version>
  </dependency>
</dependencies>
```

最后我们就可以在代码中使用它了:

```Java
char word = '虎';
System.out.println(Arrays.toString(PinyinHelper.toHanyuPinyinStringArray(word)));
```

## 使用国内镜像仓库 {id="use_china_image_repos"}

在国内下载的过程总是漫长的等待，我们可以使用阿里云镜像仓库，配置方法就是在项目的pom.xml配置文件中加入如下配置:

```xml
<repositories>
  <repository>
    <id>aliyunmaven</id>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
  </repository>
</repositories>
```



## 本地仓库和中央仓库 {id="local_and_center_repository"}

本地仓库就像是本的缓存一般，Maven 会先从本地仓库中查找，具体如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/wj9jI3MHy5C3s73aBQmw.jpg" alt="Maven repository" />

## 生命周期 {id="life_cycle"}

Maven 在不同的生命周期为我们提供了不同的命令，常用的如下表所示:

| 命令       | 用途                              |
|----------|---------------------------------|
| validate | 验证项目是否正确且所有必须信息是可用的             |
| compile  | 源代码编译在次阶段完成                     |
| test     | 运行 test 目录下的测试代码验证 src目录下源代码的逻辑 |
| package  | 生成制品，jar 或者  war 文件             |
| verify   | 运行任意的检查来验证项目包有效并且达到质量标准         |
| install  | 安装打包的项目到本地仓库，以供其他项目使用           |
| deploy   | 拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程使用  |


## 属性管理 ##

在我们的 `pom.xml` 文件中，有如下配置:

```xml
<properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	<maven.compiler.source>18</maven.compiler.source>
	<maven.compiler.target>18</maven.compiler.target>
</properties>
```

我们通过下表来解释一下上面这几项的含义:

| 属性	                          | 含义                       |
|------------------------------|--------------------------|
| project.build.sourceEncoding | 	构建时候采用的字符集编码            |
| maven.compiler.source	       | 在编译是按照 JDK 18  的语法规则进行检查 |
| maven.compiler.target	       | 生成字节码也采用 JDK 18 规则进行生成   |

上面这些属性都是 Maven 自带的属性，自然我们也可以编写自定义的属性，比如用来管理我们的依赖的版本，如下:

```xml
<properties>
	<pinyin4j.version>2.5.1<pinyin4j.version>
</properties>
```

定义了属性之后，使用示例如下:
```xml
<dependency>
	<groupId>com.belerweb</groupId>
	<artifactId>pinyin4j</artifactId>
	<version>${pinyin4j.version}</version>
</dependency>
```

## 插件 {id="plugins"}

要使用 Maven 的插件，首先我们先配置插件仓库的国内镜像，这个和之前的阿里云镜像地址是一样的，配置如下:
```xml
<pluginRepositories>
  <pluginRepository>
    <id>aliyunmaven</id>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
  </pluginRepository>
</pluginRepositories>
```

然后我们来安装并配置一个插件，这个插件可以在打包的时候将所有的 jar 合并到一个 jar 文件中:

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-assembly-plugin</artifactId>
			<version>2.5.5</version>
			<configuration>
				<archive>
					<manifest>
						<mainClass>icu.hacking.maven.PinYinPrinter</mainClass>
					</manifest>
				</archive>
				<descriptorRefs>
					<descriptorRef>jar-with-dependencies</descriptorRef>
				</descriptorRefs>
			</configuration>
		</plugin>
	</plugins>
</build>
```

使用这个插件，只需要在 IDE 中选择插件下的命令即可:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/PtfzXWiEl0r3KX4pTYih.png" alt="plugins" />

## 聚合工程 {id="project"}

关于聚合工程，需要注意下面几点:

1. 聚合工程里可以分为顶级项目(顶级工程、父项目）与子工程，这两者之间是父子关系。子工程在 Maven 中称之为模块（Module），模块之间是平级的，是可以相互依赖的。
2. 子模块可以使用顶级工程里所有的资源（依赖），子模块之间如果要使用资源，必须要构建依赖（构建关系）
3. 一个顶级工程是可以由多个不同子工程共同组合而成的

第 2 点补充说明，子模块之间如果要使用资源，必须构建依赖，其实就是在子模块中添加依赖的模块的配置:

```xml
<dependencies>
	<dependency>
		<groupId>icu.hacking</groupId>
		<artifactId>common</artifactId>
		<version>1.0-SNAPSHOT</version>
	</dependency>
</dependencies>
```

另外，如果 A 依赖了 B，B 依赖了 C，那么 A 当中是可以直接使用 C 模块的资源的。

最后，聚合工程构建完毕之后是需要使用 Maven 去 install 的:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/kAvoSUCpZ50q7d7WeO3N.png" alt="install" />

## 总结 ##

这篇文档主要描述了 Maven 的应用。它为 Java 提供了标准化的构建、打包方式。通过依赖管理简化组件之间的依赖关系，并且可以有更多额外的扩展功能。