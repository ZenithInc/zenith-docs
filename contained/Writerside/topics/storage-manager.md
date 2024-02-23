# 存储管理 

这篇文档描述了在 Docker 中管理数据的几种方式。包括如何使用 volumes, 使用 bind mounts, 使用 tmpfs, 数据卷容器，数据卷的备份以及恢复等。

## 数据保存在容器中 {id="data-in-container"}

直到目前为止，我们的数据都直接存储在 Docker 的容器当中，容器本质上就是一个进程，存储在容器中也就是存储在内存中，这会存在一些问题:

- 当容器不再运行的时候，我们无法使用数据。
- 当容器被删除的时候，数据也会跟着消失。
- 数据保存在容器的可写层中，我们无法轻松的把数据迁移到其他地方。

所以，我们需要考虑数据如何持久化的问题。

## 数据持久化 {id="data-persistence"}

Docker 提供了卷(Volumes)、挂载(Bind Mounts)、临时文件系统(Tmpfs) 这几种方式，大多数情况下，我们只会使用卷。

- Volumes: 卷存储在 Docker 管理的主机文件系统中，具体的目录是 `/var/lib/docker/volumes` ，完全由 Docker 管理。
- Bind Mounts: 绑定挂载，可以将主机上的文件或者目录挂载到容器中。
- Tmpfs: 仅存储在主机系统的内存中，而不会写入主机的文件系统。

### Volumes {id="volumes"}

我们可以使用如下命令来创建一个卷:
```shell
$ docker volume create  ## 会随机生成一个 Volume 的名字
c4dbf70166cc480b2f7e6b77da8ca0747395eba8dd1262456c9003842713ab5f    ## Volume 的名字
```
查看当前已经创建的 Volume：
```shell
$ docker volume ls
DRIVER              VOLUME NAME
local               c4dbf70166cc480b2f7e6b77da8ca0747395eba8dd1262456c9003842713ab5f
```
这种随机命名的卷，也称之为 **匿名卷** 。我们也可以在创建 Volume 的时候，手动指定名称:
```shell
$ docker volume create mysql_data
mysql_data
$ docker volume ls
DRIVER              VOLUME NAME
local               c4dbf70166cc480b2f7e6b77da8ca0747395eba8dd1262456c9003842713ab5f
local               mysql_data
```
创建了一个 Volume 之后，我们就可以在启动一个容器的时候，指定这个数据卷。当我们运行 `docker run` 命令的时候，就可以使用 `-v` 或者 `--volume` 参数来指定卷，被指定的卷可以和容器中的目录或文件做映射(即你访问容器中的该目录就等于是访问主机中的卷, 实际上，该卷是位于主机中的):
```shell
docker run -v [host-dir:]container-dir[:options]
```
命令的格式就如上所示，当中的参数我们分别来讲:

- `host-dir` ：指定主机当中的卷，这是可以省略的，如果省略的话，就会创建一个匿名卷，如果指定的是主机上的目录的话，就需要使用绝对路径。
- `container-dir` ：这个指的是挂载到容器上的目录或文件。
- `options` ：这个选项的取值范围为 `[rw|ro,][z|Z]` ，其中  `rw` 表示可读可写， `ro` 表示只读。这两种模式可以搭配 `[z|Z]` ，使用 `,`  分隔，其中 `z` 表示该卷可以被多个容器使用，而 `Z` 表示这个卷只能被当前的容器使用。

在实际的使用过程中，我们更推荐使用 `--mount` 选项来指定卷, 格式如下:
```shell
docker run --mount type=volume,src=named_volume,target=container-path,readonly=true
```
命令的格式就如上所示，当中的参数我们分别来讲:

- `type` ，挂载的类型，取值范围为 `bind` 、 `volume` 、 `tmpfs` 。
- `source` ，指的是主机管理的命名卷，也可以写成 `src` 。
- `dst`  ，也可以写成 `destination` 、  `target` 。卷在容器中映射的文件或者目录。
- `readonly` ， 取值范围为 `true` 或者 `false` ，卷是否只读。

下面我们创建一个 Volume 并创建容器使用它:
```shell
$ docker volume create test_volume			## 创建一个卷
test_volume
## 创建容器并指定卷
$ docker container run -it --name test_contianer --mount type=volume,src=test_volume,dst=/volume --rm ubuntu /bin/sh
# cd /voume     ## 可以进入卷，当我们在这个容器中往当中写入内容的时候，会同步到主机管理的卷中，不会随着容器删除而消失
```

### Bind Mounts {id="bind-mounts"}

Bind Mounts 这种形式就是将主机上的目录直接绑定到容器中去，实现在容器中对主机目录的读写。我们来实验一下:
```shell
$ sudo docker container run -it --mount type=bind,src=/home/vagrant/host-dir,dst=/docker-dir --name test2 --rm ubuntu /bin/sh
# cd /docker-dir
# touch test1		## 在容器中创建文件
# exit
$ ls			## 主机中也已经存在该文件了
test1
```

### Tmpfs {id="tmpfs"}

最后，我们演示一下使用 Tmpfs 方式，比上面两种更简单一些。当容器停止的时候，响应的数据就会被移除:
```shell
docker run -it --munt type=tmpfs,dst=/test --name test --rm ubuntu /bin/sh
```

## 使用数据卷容器共享数据 {id="data-volumes-share"}

如果容器之间需要共享一些持续更新的数据，最简单的方式就是使用用户数据卷容器。其他的容器可以挂载这个容器实现数据共享，这个挂载的数据卷的容器就叫做数据卷容器。你可以理解成是把数据卷使用容器的方式启动。

我们可以在执行 `docker run` 的时候，使用 `--volumes-from` 参数来指定数据卷容器。下面的示例中，我们创建了一个数据卷容器以及两个普通的容器，这两个普通的容器都指向了数据卷容器：
```shell
docker volume create vdata		## 创建数据卷
docker container run -it -v vdata:/vdata --name data_1 ubuntu /bin/bash  ## 创建数据卷容器
docker container run -it --volumes-from data_1 --name data_2 ubuntu /bin/bash   ## 将 data_2 指向数据卷容器 1
docker container run -it --volumes-from data_1 --name data_3 ubuntu /bin/bash   ## 将 data_3 指向数据卷容器 2
```
这样我们就实现了两个容器共享一个数据卷容器。

## 备份数据卷容器中的数据 {id="backup-volumes"}

数据存储于数据卷中，如果我们想要备份它，可以采用创建备份容器的方式:
```shell
$ docker container run --volumes-from data_1 -v /home/vagrant/backup:/backup ubuntu tar cvf /backup/backup.tar /vdata/
/vdata/
/vdata/test.t
tar: Removing leading `/` from member names
$ ls /home/vagrant/backup/
backup.tar			## 备份的文件
```
那么如何恢复文件呢？应该也是一样的道理吧：
```shell
$ docker container run --volumes-from data_1 -v /home/vagrant/backup:/backup ubuntu tar xvf /backup/backup.tar -C /
vdata/
vdata/test.t
```
启动一个新的恢复容器，然后将数据解压到 `vdata` 的上一级目录中去。