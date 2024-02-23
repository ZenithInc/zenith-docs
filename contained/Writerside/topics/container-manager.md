# 容器管理 

这一篇文档描述了容器管理的一些命令，比如启动容器、退出容器、关闭容器、删除容器等。容器的管理使用 `docker contaner` 命令。

### 基本操作

我们从 Hello World 开始，如何创建一个容器呢？

```shell
docker container run hello-world
```

`docker run` 是一条复合命令，严格来说并不是创建容器的命令。创建容器使用 `docker create` 命令，在说它之前先来说说 `docker run` 都做了什么事情？

1. 检查本地是否存在指定的镜像
2. 如果不存在指定的镜像，则连接 Docker Registry 下载
3. 下载之后创建容器
4. 创建容器之后运行容器, 启动容器可以用 `docker container start` 命令

`docker run` 有一些常用的参数，如下:

- `-i` 或者 `--interactive` : 交互模式
- `-t` 或者 `--tty` ：分配一个 `pseduo-TTY` ，伪终端
- `--rm` ：在容器退出后自动移除
- `-p` ：将容器的端口映射到主机
- `-v` 或 `--volume` ：指定数据卷

我们也可以在后面加上容器运行后要执行的 Shell：

```shell
docker container run busybox echo "hello world"
```
如果我们希望容器保持运行，可以指定 `-it` 参数:

```shell
$ docker container run -i -t ubuntu /bin/bash
root@14b5c7524f2f: /#
```
默认情况下，主机名 `14b5c7524f2f` 是容器的 ID。那么我们如何退出容器呢？可以在终端输入 `exit` ，然后使用如下命令观察容器的状态:
```shell
docker container ls -a # 你可以看到状态为 Exited
```
我们也可以指定 `-d` 参数，让容器在后台运行:
```shell
docker container run -itd ubuntu /bin/bash
## 观察容器状态，应该为 Up
docker container ls		## 如果不加上 -a, 只显示运行状态的容器
```
然后，我们再来看看 `docker create` 命令，相对于 `docker run` 只会创建容器，但是不会运行。执行后返回容器的 ID:
```shell
docker container create --name shiyanlou \		## 指定容器名称
	--hostname shiyanlou \											## 指定 Hostname
  --mac-address 00:01:02:03:04:05 \						## 指定 mac 地址
  --ulimit nproc=1024:2048 -it ubuntu /bin/bash		## 指定 ulimit
## 观察容器状态
docker container ls  ## 应该是 Created
```
查看 docker 容器的详细信息可以使用如下命令:
```shell
docker container inspect [容器名称|容器ID]
```

## 容器的的运行模式 {id="container-run-mode"}

容器有三种运行模式，分别是 `attached` 模式、`detached` 模式以及可交互模式。下文分别予以介绍。

### Attached 模式 {id="attached"}

当你使用`docker container run` 命令来运行一个容器之后，容器就处于 `attached` 模式，容器中的输出会输出到当前的输出中，同样当前环境中信号的输入也会传递到容器中去。比如你使用`CTRL+C` ，信号也会传递给容器，从而停止容器进程的运行。
> 注意: Windows 下并不支持完整的 Attached 模式，并不支持信号向容器的传递。所以即是你使用`CTRL+C`，也不会终止容器进程的运行。


### Detached 模式 {id="detached"}

如果你在`docker container run` 命令的后面使用 `-d` 参数，那么容器就会进入 `detached`模式运行。成为一个后台进程，并不会输出容器中的输出内容，也不会传递信号。比如你可以尝试如下的命令:

```shell
$ sudo docker container run -d -p 80:80 nginx
c10483e35e0144401750b02c37ff72f047a4048357aa49a76a2b83b487a4a618
## 我们也可以通过 docker 的 attach 命令将容器从 detached 模式切换到 attached 模式
$ sudo docker attach c1048
```

在 `detached` 模式下，我们可以通过`docker container log` 来查看指定容器中的输出:

```shell
$ sudo docker container logs c104
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
```

使用 `-f` 参数可以动态的查看日志，类似于 `tail -f` Linux 命令。

### 可交互模式 {id="it-mode"}

经常我们可能需要进入容器内部查看一些信息或者做某些操作，这时候我们就可以使用可交互模式来运行容器。如下示例:
```shell
$ sudo docker container run -it ubuntu sh
# 省略部分输出内容	
# ls
bin  boot  dev	etc  home  lib	lib32  lib64  libx32  media  mnt  opt  proc  root  run	sbin  srv  sys	tmp  usr  var
```
加上 `-it` 参数就进入了交互模式，可以在最后面加上要在容器中执行的命令。如果执行的是`sh` 命令，就可以在运行的`sh` 中，执行连续的命令。

对于一个已经在运行中的容器来说，我们可以使用`exec` 子命令来进入交互模式，如下所示:
```shell
$ sudo docker container run -d nginx
a170710178b2b89f29fa3b362bb65737ae5bbd8ed88a3542d31fb902c747d272
$ sudo docker container exec -it a17 sh
# ls
bin   dev		   docker-entrypoint.sh  home  lib64  mnt  proc  run   srv  tmp  var
boot  docker-entrypoint.d  etc			 lib   media  opt  root  sbin  sys  usr
```

## 更多的操作 {id="more-actions"}

更多的操作如下面表格所示:

| 操作                       | 命令                                             |
|--------------------------|------------------------------------------------|
| docker container stop    | 停止容器                                           |
| docker container restart | 重启容器                                           |
| docker container pause   | 暂停容器                                           |
| docker container unpause | 恢复暂停的容器                                        |
| docker container attach  | 连接到后台运行的容器的终端                                  |
| docker container logs    | 查看指定容器的日志，一般配合参数 `-t` 和 `-f` 使用，前者显示时间戳，后者实时输出 |
| docker container top     | 查看容器的进程信息                                      |
| docker container diff    | 查看容器中的文件修改                                     |
| docker container rm      | 删除容器, 不能删除正在运行的容器，除非使用`-f`参数                   |

上面的一些命令可以批量操作，比如说`stop`、`rm` 等。批量操作如下:
```shell
docker container stop $(docker container ps -qa)
```
其中, `-q` 表示只显示容器的 ID， 而 `-a` 表示显示所有的容器，包括哪些已经退出的。我们可以使用下面的指令，快捷删除所有已经退出的容器：
```shell
$ docker system prune -f
Deleted Containers:
8e45a9612cc11a678114565770a0e2d4583832f40ccbf0aa39290caa70ce34d4
7b85a7987e1a18e58639a8c7735666ee414ff8d37bcf0477fd6f8fd343c2c414

Total reclaimed space: 3.279kB
```
