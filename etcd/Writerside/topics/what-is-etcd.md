# 什么是 ETCD 

一个名为 CoreOS 的创业团队，为了构建一个名为 Container Linux 的开源、轻量级的操作系统，可以自动化、快速部署应用程序。这就需要一个协调服务，在对比了众多开源服务之后，最后选择自研，这就有了 ETCD。

## ETCD 简介 {id="introduction"}

在文档开头我们已经说明了 ETCD 的由来，那么它到底是什么呢？你可以打开 [ETCD 官网](https://etcd.io/)看看它的描述:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/2QhawiRXOy2yy5pum6wa.png" alt="etcd"/>

它是一个分布式的键值存储系统，为在分布式系统中存储最为关健的数据而生。

它这个名为 CoreOS 的团队最初也考虑了 ZooKeeper，但是因为使用 Java 编写，部署麻烦、占用内存高，以及使用自有 RPC 协议，无法简单与 CURL 交互。所以，才有了 ETCD，它使用 Go 语言编写，目前最新版本是 [v3.5](https://etcd.io/docs/v3.5/quickstart/)。

## ETCD 名字的由来 {id="how-etcd-got-its-name"}

看着这个名字是不是有点眼熟？但是又想不起来在哪里看过？在 Unix/Linux 中看过：

```Shell
cd /etc
```

在 like-unix 的操作系统中，`/etc/` 目录一般用于存储配置文件信息，这也说明 etcd 的核心作用，后面的 `d` 是单词 `distribute` 的缩写，表示是分布式的。所以 etcd 就表示其是用于存储分布式的配置存储服务。