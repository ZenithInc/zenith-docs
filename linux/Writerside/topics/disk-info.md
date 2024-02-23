# 查看磁盘信息 

这一篇文档主要描述了在 Linux 操作系统中，对磁盘进行有效的管理。

## df 命令 {id="df"}

在日常的实践中，我们经常需要对磁盘的使用情况进行查看，我们可以使用`df`命令,如下示例:
```shell
$ df
文件系统          1K-块    已用     可用 已用% 挂载点
devtmpfs         923188       0   923188    0% /dev
tmpfs            936620      24   936596    1% /dev/shm
tmpfs            936620     396   936224    1% /run
tmpfs            936620       0   936620    0% /sys/fs/cgroup
/dev/vda1      51539404 4760728 44581432   10% /
tmpfs            187324       0   187324    0% /run/user/0
```
> 使用了`-h`选项，可以自动以合适的单位显示，例如(1K, 50M, 100G)。

我们也可以具体指定某一磁盘，如下：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/aBqWmCGDZu49KYjRrCTn.png" alt="df command"/>


## du 命令 {id="du"}

与 `df` 命令不同的是，`du` 命令可以对指定的目录或文件的使用情况进行查看。

如果不携带任何参数，则显示的当前目录的空间使用情况，如下:
```shell
$ du | head -n 5
8	./.ssh
8	./.acme.sh/hk.hacking.icu/backup
28	./.acme.sh/hk.hacking.icu
16	./.acme.sh/ca/acme.freessl.cn
12	./.acme.sh/ca/acme.certcloud.cn
```

可以指定多个文件，统计所占用的空间:
```shell
$ du content.txt workspace/ | head -n 5
4	content.txt
20	workspace/vue-element-admin/mock/role
48	workspace/vue-element-admin/mock
128	workspace/vue-element-admin/.git/hooks
8	workspace/vue-element-admin/.git/logs/refs/remotes/origin
```

默认情况下，会递归查询目录中所有的文件的大小，我们可以使用`--max-depth`参数来指定查询目录的层次，例如`du -h --max-depth=1`。

## 实时查看 IO 信息 {id="io-info"}

我们可以使用 `iostat`这条命令来查看 CPU 的统计信息以及整个系统、适配器、终端设备和硬盘的输入/输出信息。如下:
```shell
iostat
              disk0       cpu    load average
    KB/t  tps  MB/s  us sy id   1m   5m   15m
   41.00    4  0.18   3  1 96  1.39 1.52 1.62
```
我们着重来看第三部分磁盘信息的输出,展示了各个存储设备上的 I/O 情况，包括块的读写量和读写速度等。

当我们怀疑磁盘出现性能问题的时候，需要查看磁盘的各种信息以印证自己的判断。比如磁盘的工作状态是否正常？TPS 是多少？吞吐量有没有问题？我们可以使用`iostat -d -k 1 3`这条命令,`-d`表示自显示磁盘的使用状态，并不包含 CPU 信息,`-k`选项表示使用  KB 作为单位。`1 3`分别表示采样时间为 1 秒，采样次数为 3 次，输出如下所示:
```shell
iostat -d 1 3
              disk0
    KB/t  tps  MB/s
   41.00    4  0.18
    0.00    0  0.00
    6.86    7  0.05
```
输出的字段的解释如下表所示:

| 字段        | 描述                |
|-----------|-------------------|
| tps       | 每秒进程的 I/O 读、写请求总数 |
| kB_read/s | 每秒读取的字节数(单位为 KB)  |
| kB_wrtn/s | 每秒写入的字节数(单位为 KB)  |
| kB_read   | 读取的字节总数(单位为 KB)   |
| kB_wrtn   | 写入的字节总数(单位为 KB)   |


使用 iostat 命令的输出中的第一组数据，相对其他组数值要大出很多。这是因为这组数据是表示 Linux 系统启动到本命令执行这段时间的统计结果，而后面的数据都指的是一个采样周期内的统计。

我们还可以利用 iostat 命令来显示更多有关于磁盘的信息,比如使用`iostat -d -x -k 1 3`, `-x`选项展示了更多的硬盘统计数据，截图如下:

> 注意: 此处缺少图片。


截图中输出的各个字段的含义如下表所示:

| 字段       | 描述                                                                                        |
|----------|-------------------------------------------------------------------------------------------|
| rrqm/s   | 每秒对该设备的读请求被合并次数，文件系统对读取同块(block)的请求进行合并                                                   |
| wrqm/s   | 每秒对该设备的写请求被合并次数，文件系统对写入同块(block)的请求进行合并                                                   |
| r/s      | 每秒完成读 I/O 的次数                                                                             |
| w/s      | 每秒完成写 I/O 的次数                                                                             |
| rkB/s    | 每秒读千字节数                                                                                   |
| wkB/s    | 每秒写千字节数                                                                                   |
| avgrq-sz | 平均每次 I/O 读写的数据大小(扇区)                                                                      |
| avgqu-sz | 平均等待处理的 I/O 请求队列长度                                                                        |
| await    | 平均每次 I/O 请求等待时间(包括等待时间和处理时间，单位为毫秒)，可以理解为 I/O 的响应时间。一般系统的 I/O 响应时间应该低于 5ms，如果大于 10ms 就比较大了 |
| r_await  | 平均每次读请求等待时间                                                                               |
| w_await  | 平均每次写请求等待时间                                                                               |
| svctm    | 平均每次 I/O 操作的服务时间，单位为毫秒                                                                    |
| %util    | 周期内用于 I/O 操作的时间比率，即 I/O 队列非空的时间比率,即(r/s + w/s)*(svctm/1000)                               |


我们如何来查看这些参数来断定问题呢？如下：

- 如果 %util 较大，则代表 I/O 请求太多，硬盘可能存在瓶颈。
- await 大于 svctm，差值越小，则说明队列时间越短；反之，则队列时间越长，说明系统出了问题。
- 如果 svctm 比较接近 await,则说明 I/O 几乎没有等待时间。
- 如果 await 远大于 svctm，则说明 I/O 队列太长，应用响应时间也变长。
- avgqu-sz 队列长度也可亨利那个 I/O 负荷的指标，avgqu-sz 是单位时间内的平均值。
