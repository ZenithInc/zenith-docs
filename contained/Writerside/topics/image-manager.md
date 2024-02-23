# 镜像管理 

什么是镜像呢？镜像就是一个模版，用来创建容器的模板。当我们执行“docker run”的时候，会首先查找本地是否存在对应的镜像，如果不存在则会在 Docker Registry 上查找下载。Docker 的镜像是增量的修改，每次创建新的镜像都会在老的镜像上面构建一个增量的层(Layer)，有利于层的复用，减少镜像的体积，利于网络传输。

我们先来说几个镜像相关的概念，比如说 Repository 是镜像存储的位置，名为仓库。而 Registry 是镜像仓库的注册服务器，每个仓库中都包含了很多的镜像。每个镜像还会又一个 Tag（标签), 这个 Tag 我们一般会标记为镜像的版本，这点就 Git 中的版本的概念是一致的。比如说 Ubuntu:14.04 ，其中 Ubuntu 就是镜像的名字，而 14.04 是它的 Tag，也是它的版本。

## 镜像的基本操作 {id="basic"}

然后我们可以使用 `docker image ls` 命令来查看本地的镜像:
```Shell
$ docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    bf756fb1ae65   14 months ago   13.3kB
```

指定镜像的名称也是可以的， 指定版本可以使用 `镜像名字:版本`：
```Shell
$ docker image ls hello-world
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    bf756fb1ae65   14 months ago   13.3kB
```

如果要查看一个镜像的详细信息，如下:
```Shell
$ docker image inspect hello-world | grep Id   ## 输出太多的内容，过滤一下
"Id": "sha256:bf756fb1ae65adf866bd8c456593cd24beb6a0a061dedf42b26a993176745f6b",
```

联网搜索镜像使用如下命令:
```Shell
$ docker search ubuntu | head -n2  ## 输出太多的内容，过滤一下
NAME     DESCRIPTION     STARS     OFFICIAL   AUTOMATED
ubuntu   Ubuntu is a …   11992     [OK]
```

拉取镜像使用如下命令:
```Shell
$ sudo docker image pull ubuntu:14.04
14.04: Pulling from library/ubuntu
2e6e20c8e2e6: Pull complete
## ...省略部分输出内容...
```

对于拉取下来的镜像，默认的存储路径如下:
```Shell
$ sudo ls /var/lib/docker/overlay2  ## 省略输出内容
```

## 镜像的创建 {id="create-image"}

创建新的镜像有两种方式，一种方式是修改已经存在的容器，在这个基础上创建镜像，另一种方式是使用 Dockerfile 文件来创建镜像。在生产环境下，我们推荐使用第二种方式来创建镜像。

### 修改已经存在的容器创建镜像 {id="modify-exist-image"}

如果我们要创建一个新的镜像，可以基于一个已经存在的镜像。用它来创建一个容器，然后在容器中进行修改，之后提交到一个新的镜像中去:
```Shell
$ sudo docker run -it --name test busybox /bin/sh
## ...省略部分输出内容...
/ # touch test1 test2			## 在容器中创建两个新的文件
## 使用 CTRL+P 以及 CTRL+Q 退出容器
```
在退出容器之后，我们使用 `docker container commit` 命令来创建一个新的镜像:

```Shell
$ docker container commit test new_test			## 由当前运行着的 test 容器创建出一个新的镜像，名为 new_test
sha256:6c88835118a947a1d2af0313157bb0a55204299f9ff4096297fefabe18a987e2
```

但是这种方法并不建议在生产环境中使用，因为这样创建出来的镜像非常难以维护。我们推荐使用 Dockerfile 的方式来创建一个新的镜像。

### 使用 Dockerfile 创建镜像 {id="use-dockerfile"}

Dockerfile 文件是一个用来描述镜像的创建过程的文件，换句话说我们通过 Dockerfile 这个文件告诉 Docker，我们这个镜像的一些基本信息，如何创建这个镜像，创建这个镜像之后要做什么事情。怎么告诉呢？通过预定义的一些指令,指令的格式如下:
```Shell
INSTRUCTION arguments   ## 它是有指令和参数构成的，就和 Linux 下的命令是一样的逻辑
```

那么一个基本的 Dockerfile 都包含那些内容呢？其实这个问题的答案也就是我们之前提的几个问题的答案，如何创建这个镜像？创建之后要干什么？
```Docker
FROM ubuntu:14.04
MAINTAINER JinZhiSu/happy@hacking.icu
RUN apt-get update && apt-get install apache2
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]		## CMD 只能出现一次，如果出现多次，只会执行最后一条命令
```
解释一下这段内容:

1. 基础镜像：我们大概不会从零创建一个镜像，我们需要一个以某个镜像为基础，在这个基础上增加我们自己的内容，使用 `FROM` 指令来指定。
2. 维护者信息: 这个镜像是谁维护的，除了问题找谁？通过 `MAINTAINER` 指令可以指定维护者的名字和邮箱。
3. 镜像操作命令：我们要在基础镜像上进行那些修改？是创建文件还是安装服务器环境或者进行特殊的配置？常用的命令是 `RUN` ，运行 Shell 的命令或脚本。
4. 容器的启动命令：当容器启动的时候需要执行那些命令或脚本？使用指令 `CMD` 或者 `ENTRYPOINT` 。

通过短短 4 行，我们就创建了一个新的 Dockerfile, 也就创建了一个新的镜像。但是，我们还需要让这个 Dockerfile 通过 docker 成为真正的镜像文件:
```Shell
$ docker image build .		## . 表示当前目录，会自动查找当前目录下的 Dockerfile, 当然我们也可以使用 -t 参数来指定具体的 Dockerfile 文件
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM ubuntu:14.04
# ...省略更多输出...
```

然后我们可以通过 `docker image ls` 命令来查看刚刚创建的 Docker 镜像:

```Shell
$ docker image ls                                                                    [15:42:04]
REPOSITORY TAG     IMAGE ID            CREATED              SIZE
<none>    <none>   2d59468593eb        About a minute ago   197MB
```

## 删除镜像 {id="delete"}

删除一个镜像可以使用如下命令:
```Shell
$ docker container rm 956		## 不能删除已经被容器使用的镜像
956
$ docker image rm 2d5
Deleted: sha256:2d59468593ebb585aa82cd02989cada50c3145979858801555115c358a1af8d5
```

如果要批量删除没有使用的镜像，可以如下示例:
```Shell
docker image prune -a
```

## 镜像的导入和导出 {id="import-and-export"}

当我们在内网或者其他特殊的网络环境下，无法连接到 Registry 的时候，就可以通过将已经存在的镜像导出到存储设备中，然后在对应的环境中部署。

镜像的导出如下:
```Shell
$ docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
ubuntu       latest    c29284518f49   7 days ago   72.8MB
$ docker image save ubuntu -o ubuntu.image
$ ls
ubuntu.image
```

导入镜像如下所示:
```Shell
$ docker image load -i ./ubuntu.image
a70daca533d0: Loading layer [================>]  75.16MB/75.16MB
Loaded image: ubuntu:latest
```

## 运行 MySQL 容器 {id="run-mysql-container"}

关于 MySQL 的镜像，我们可以参考[官方文档](https://hub.docker.com/_/mysql)。如果环境变量比较多，我们可以创建一个文件用来保存，而不是通过命令的方式指定:
```text
MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=test
MYSQL_USER=shiyanlou
MYSQL_PASSWORD=Xbcd20198$
```

然后我们来运行 MySQL 的容器：
```Shell
docker run --name mysql -d --env-file ./env_file -p 3306:3306 mysql:5.5
```

## 总结 {id="summary"}

这篇文档我们描述了如何操作镜像，包括创建、查看、删除等操作。