# ss

在很长一段时间内，我们都使用 netstat 命令来查看网络的连接信息。现在，有一个俄罗斯人，名字叫做 Alexey Kuznetsov。他写了一款软件，名字叫做 ss，相对前者，它有着更快的速度。所以，我们应该来学学这个 ss 怎么使用。

## 安装 {id="install"}

我们使用的 Linux 上不一定安装了 ss。如果安装了，我们可以使用如下命令来查看其所在的安装包:

```shell
rpm -qf /usr/sbin/ss ## 输出: iproute-4.11.0-14.el7.x86_64
```

如果没有安装,我们可以使用`sudo yum install iproute -y`命令来安装它。

## 基本使用 {id="usage"}

下面展示了一些基本的使用案例。

### 查看当前服务器的网络连接统计

使用命令`ss -s`,输出信息如下截图:
```shell
$ ss -s
Total: 192
TCP:   6 (estab 2, closed 0, orphaned 0, timewait 0)

Transport Total     IP        IPv6
RAW	  0         0         0
UDP	  3         2         1
TCP	  6         4         2
INET	  9         6         3
FRAG	  0         0         0
```
关于 TCP 中的连接状态解释如下:

| 连接状态     | 描述                          |
|----------|-----------------------------|
| estab    | 已经建立 TCP Socket 的数量         |
| closed   | 初始状态，表示没有任何连接[1]            |
| orphaned | 没有用户态 fd 的 TCP Socket 数量[2] |
| synrecv  | 处于 SYN_RECV 状态的 TCP         |
| timewait | 处于 TIMEWAIT 状态的 TCP         |


- [1] 处于`CLOSE_WAIT`、`LAST_ACK`、`TIME_WAIT`状态的 TCP Socket 数量
- [2] 处于`FIN_WAIT_1`、`FIN_WAIT_2`、`CLOSING`状态的 TCP Socket 数量。在用户态程序发送数据并关闭 fd,若此时这个 fd 关联的 Socket 还没有将缓冲区内的数据全部发出去，则在内核中存在一个 orphaned socket。

### 查看所有打开的网络端口

使用命令`ss -l`,输出如下截图:


| 输出列               | 描述                                            |
|-------------------|-----------------------------------------------|
| Recv-Q            | 表示当前等待服务器端掉用 accept 完成三次握手的 listen backlog 数量 |
| Send-Q            | 未知                                            |
| LocalAddress:Port | 本地的端口                                         |
| Peer Address:Port | 远程的端口                                         |


使用`-p`选项可以输出和具体的程序的名称。

### 查看服务器上所有的 Socket 连接

具体的参数组合如下表:

| 命令     | 描述            |
|--------|---------------|
| ss -a  | 列出所有的网络连接     |
| ss -ta | 列出所有的 TCP 连接  |
| ss -ua | 列出所有的 UDP 连接  |
| ss -wa | 列出所有的 RAW 连接  |
| ss -xa | 列出所有的 Unix 连接 |

