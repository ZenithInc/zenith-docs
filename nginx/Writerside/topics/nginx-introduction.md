# Nginx 简介 

这篇文档介绍了什么是 Nginx，其特点和优势是什么，以及如何安装部署。并且介绍了什么是反向代理、基础的命令行操作。

## 什么是 Nginx {id="what-is-nginx"}

什么是 Nginx? 我们来看 《[nginx” in “The Architecture of Open Source Applications](https://www.aosabook.org/en/nginx.htm)》一文中的相关简介，其中文翻译版本《[[译] 开源项目之 Nginx](https://blog.csdn.net/weixin_33774615/article/details/91450578)》，其原文节选如下:

> nginx (pronounced "engine x") is a free open source web server written by Igor Sysoev, a Russian software engineer. Since its public launch in 2004, nginx has focused on high performance, high concurrency and low memory usage. Additional features on top of the web server functionality, like load balancing, caching, access and bandwidth control, and the ability to integrate efficiently with a variety of applications, have helped to make nginx a good choice for modern website architectures. Currently nginx is the second most popular open source web server on the Internet.


Nginx 是由俄罗斯软件工程师 Igor Sysoev 开发的一款免费开源 Web 服务器。自 2004 年公开发布以来，Nginx 主要专注于提供高性能、高并发以及低内存占用。除了基本的 Web 服务器功能外，Nginx 还提供诸如负载均衡、缓存、访问控制和带宽控制等功能，以及与各种应用程序高效集成的能力。这些特性使得 Nginx 成为了现代网站架构的一个好选择。目前，Nginx 已成为互联网上第二大流行的开源 Web 服务器。

## 特点和优势 {id="feature"}

Nginx 有下面这些优势特性

- 高并发、高性能
- 扩展性好，有着非常多的模块和生态
- 异步非阻塞的事件驱动模型(是高并发、高性能的原因)
- 高可靠性
- 热部署、平滑升级
- BSD 许可证，开源，可商业应用
- 大厂背书, 比如 Google，阿里，腾讯，百度......

下面这张图显示了 Nginx 截至当前的一个市场占有率:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/uRmsIovgnK7tfXodDRE1.png" alt="Web server developers: Market share of all sites"/>

截图来源于: [news.netcraft.com](https://news.netcraft.com/archives/category/web-server-survey/)

## 模块化体系 {id="modules"}

下图展示了 Nginx 的模块化体系，因为模块化，所以可以方便扩展:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/1HXrjeNRFZymXusb3DKC.png" alt="nginx module"/>


## 安装 Nginx {id="how-to-install"}

在 Nginx 官网的下载页面，提供了三种类型的版本下载，如下截图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/FWnl6sfeRQoPsDvtYl6T.png" alt="nginx versions"/>

Mainline 指的是正在开发的版本，可能并不稳定。而 Stable 指的是稳定发布版本，一般使用这个版本即可。而 Legacy 指的是历史版本。

### 使用 Docker 安装 {id="docker-installation"}

使用 Docker 安装:

```shell
docker run --name some-nginx -v /some/content:/usr/share/nginx/html:ro -d nginx
```

### Yum Repo {id="yum-installation"}

使用 Yum 安装, 创建脚本如下：

```shell
#!/bin/bash
## 配置 Yum 仓库
sh -c 'cat <<EOF > /etc/yum.repos.d/nginx.repo
    [nginx-stable]
    name=nginx stable repo
    baseurl=http://nginx.org/packages/centos/\$releasever/\$basearch/
    gpgcheck=1
    enabled=1
    gpgkey=https://nginx.org/keys/nginx_signing.key
    module_hotfixes=true
EOF'
## 安装
yum install nginx -y
nginx -v   ## 输出版本号
## 启动服务并设置开机启动项
systemctl start nginx
systemctl enable nginx
```

是否启动成功可以访问服务:

```shell
curl -XGET
```

其他的系统或发行版，可以参考官方文档: [Installing nginx](https://nginx.org/en/docs/install.html)。

## 什么是反向代理 {id="what-is-reverse-proxy"}

既然有反向代理(the Reverse Proxy Server)，就会有正向代理，两者都是代理，只是方向不同。

那什么是代理呢？比如我要访问 Google，因为政治环境，我们没办法通过正常访问。于是我通过先访问香港的VPN服务器，然后通过VPN服务器**代替**我去访问 Google,然后将内容返回给我。这就是代理，而且是正向代理。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/7vxT9rM924vE2KXg7xkq.png" alt="proxy"/>

正向代理（Forward Proxy）是客户端和目标服务器之间的中介。客户端连接到正向代理服务器，并通过它来请求目标服务器上的资源。目标服务器只看到来自代理服务器的请求，而不是客户端本身的请求。

在这种情况下，正向代理为客户端提供了几个功能：

1. **隐藏客户端身份**：由于请求似乎来自代理服务器，因此客户端的身份（如IP地址）对目标服务器是不可见的。
2. **内容过滤**：代理服务器可以对客户端的请求进行审查，根据特定的策略（如公司或学校的上网策略）来拒绝或允许访问某些资源。
3. **缓存服务**：正向代理可以缓存请求的内容，当后续请求相同内容时，可以直接从缓存中提供，减少延迟和网络带宽消耗。
4. **访问地域限制内容**：如果代理服务器位于可以访问某些地域限制内容的地区，客户端可以通过代理服务器访问这些内容。
5. **记录用户请求**：出于监控或审计的目的，可以记录所有经过代理服务器的客户端请求。


而反向代理呢？是 Google 在收到我的请求之后，通过其代理服务器将请求转发给提供搜索服务的服务器集群，有他们中的一台服务器实例提供给我想要的内容。相对于正向代理，这种就叫做反向代理。

反向代理的出处是一片 Sun 公司的文章 ———— 《[Securing Web Applications through a Secure Reverse Proxy](https://www.informit.com/articles/article.aspx?p=169534)》。这篇文章大致讲述了使用设置反向代理服务器，集中客户端的请求，然后在转发给后端服务器集群，从而来保障后台服务器集群的安全。

从这个角度而言，反向代理指的就是负载均衡器, 而 Nginx 就提供了负载均衡的能力。**反向代理只是相对于正向代理而言的，代理才是其本质**。

## 常见的应用服务器 {id="web-server"}

| MS IIS         | 应用于 asp.net 应用    |
|----------------|-------------------|
| Weblogic、JBoss | 传统行业 ERP、物流、电信、金融 |
| Tomcat、Jetty   | J2EE              |
| Apache、Nginx   | 静态服务、反向代理         |
| Netty          | 高性能服务器编程          |

## Nginx 命令行解析 {id="cli"}

启动 Nginx 可以使用如下命令:

```shell
$ nginx -c <config_path>
```

正确退出 nginx:

```shell
$ nginx -s quit  # 在有对外服务的情况下，不要使用 stop
```

检查 nginx 配置是否正确:

```shell
$ nginx -t
```

重新加载 nginx 配置文件:

```shell
$ nginx -s reload
```

## Nginx 的作者和轶事 {id="nginx-author"}

Nginx 的作者是俄罗斯的伊戈尔·赛索耶夫([Igor Sysoev](https://en.wikipedia.org/wiki/Igor_Sysoev))，毫无疑问他是一个人才。Nginx 这个开源项目有着良好的结构和优雅的代码，简直是有口皆碑。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/2yykoH8FTBb0eEhCqn2B.png" alt="nginx author:Igor Sysoev"/>

不仅如此，其高性能和良好的生态与让 Nginx 在 2019年2月成为了互联网上部署最广泛的服务器，超越了 Apache 的 Httpd。

但是 Nginx 其实是作者在上班期间做的私活，所以后面有发生了版本之争，引申除了程序员写的私活其版权归属的问题。有兴趣的同学，可以了解一下: [Russian police raid NGINX Moscow office](https://www.zdnet.com/article/russian-police-raid-nginx-moscow-office/)。

## Nginx 在线资源 {id="nginx-online-resources"}

介绍几个非常有用的网站:

1. [Nginx 官网](https://nginx.org)
2. [Nginx 在线配置](http://file-linker.oss-cn-hangzhou.aliyuncs.com/2yykoH8FTBb0eEhCqn2B.png)
3. [Nginx Playground](https://nginx-playground.wizardzines.com)