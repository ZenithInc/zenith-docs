# 网络管理 

在这一篇文档中，你讲看到 Docker 中如何进行网络的管理，包括网络的类型以及它们的应用。

## 默认的网络类型 {id="network-type"}

当我们安装好 Docker 之后，就已经创建了三个默认的网络，可以使用如下命令查看：
```shell
$ sudo docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
dff28c734f9f   bridge    bridge    local
1e7467b6d95b   host      host      local
2ea4d9c5d1fb   none      null      local
```
下面我们分别来介绍和演示这三种网络类型。我们可以通过如下命令来查看网络的详细信息:
```shell
$ sudo docker network inspect bridge | grep Name		## 过滤掉除名字外其他的内容
        "Name": "bridge",
```

## Bridge {id="bridge"}

Bridge 是桥接网络, 默认情况下，我们创建的容器如果没有指定网络（使用 `--network` 参数指定）使用的就是桥接网络。**桥接网络的特点是，容器之间是可以相互通信的，但是容器和主机是不能联通的，需要通过 iptables 来实现端口映射。**
```shell
$ docker run -itd --name test_1 ubuntu:16.04 /bin/sh		## 创建第一个容器
$ docker run -itd --name test_2 --network bridge ubuntu:16.04 /bin/sh  ## 创建第二个容器
```
然后我们分别查看两个容器的 IP 以及子网:
```shell
$ docker inspect test_1 | grep -E "IPAddress|Gateway|IPPrefixLen" | head -n4
            "Gateway": "172.17.0.1",
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
$ docker inspect test_2 | grep -E "IPAddress|Gateway|IPPrefixLen" | head -n4
            "Gateway": "172.17.0.1",
            "IPAddress": "172.17.0.3",
            "IPPrefixLen": 16,
```
我们看到这两个容器都在同一个网段内，你可以使用 `docker attach` 来验证网络是否互联:
```shell
## 默认情况下，这个 Ubuntu:16.04 没有安装 ping，安装如下
apt update
apt install iputils-ping
## 安装完成之后
ping 172.12.0.3 ## test_2 容器也是一样的过程
```
我们前面也说到了，如果要在主机上访问容器中的服务，是通过 iptables 实现端口映射的。我们来做一个实验，创建一个 Apache 的容器，Dockerfile 如下:

```Docker
FROM ubuntu:14.04
MAINTAINER JinZhiSu/happy@hacking.icu
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get -yqq update && apt-get -yqq install dialog && apt-get install -yqq apache2
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

然后构建镜像，并创建容器:

```shell
docker image build -t apache:1.0 .
docker run -d -p 10001:80 --name web apache:1.0
```
然后查看 iptable 的规则，你可以看到 `DNAT` 的规则发生了变化:
```shell
$ sudo iptables -t nat -nvL
## ...省略部分内容...
5   320 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:10001 to:172.17.0.2:80
## ...省略部分内容...
```

## 自定义网络 {id="custom-network"}

默认的 bridge 网络，每次重启容器，容器的 IP 都会发生变化。对于默认的 bridge 网络，不能在启动容器的时候指定 IP。为了达到这个目的，我们可以使用自定义网络。在旧版的 Docker 中，通常通过 `--link` 参数来实现容器互联，但是现在已经不推荐这么做了，所以就不讲了。

首先要创建一个自定义的网路，设定驱动为 `bridge` 桥接模式:
```shell
$ sudo docker network create my-net
9095a08fdf7d071da19861467665d4d323d5696baa5b9cf416c0ca591499e3f4
$ sudo docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
9095a08fdf7d   my-net    bridge    local			## 我们自己创建的网络
```
然后创建两个容器，全部指定网络为刚才我们创建的这个自定义网络:
```shell
$ sudo docker run -itd --name test3 --network my-net ubuntu:14.04 /bin/sh
$ sudo docker run -itd --name test4 --network my-net ubuntu:14.04 /bin/sh
$ sudo docker attach test3
# ping test4			## 可以使用容器名字替换 IP
PING test4 (172.18.0.3) 56(84) bytes of data.
64 bytes from test4.my-net (172.18.0.3): icmp_seq=1 ttl=64 time=0.106 ms		## 证明容器之间可以相互联通
```

## Host 网络 {id="host"}

Host 网络指的是容器和主机可以互联。举个例子来说, 我们可以通过容器采用 Host 网络实现，将容器中的服务端口直接注册到本地，就如同本地创建了这个服务一般:
```shell
$ docker container run --name web -d --network host nginx
3bab7527ac3855dcd13bfb4d9e2f21dce1ba6f48883c54d5a99f4d10e6dfb42c

$ curl localhost
......输出省略
<h1>Welcome to nginx!</h1>
......输出省略
```

## None 网络 {id="none"}

none 网络指的是在容器中不提供其他网络接口。none 网络的容器创建之后还可以自己 connect 一个网络，比如使用 `docker network connect bridge 容器名` 可以将这个容器加入到 bridge 网络中。
