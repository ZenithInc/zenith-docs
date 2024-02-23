# ping

在实践中，我们经常使用 ping 命令来测试网络是否联通。比如，我们测试本地是否能够联通百度，可以使用`ping www.baidu.com`。这条命令十分简单，同时也十分的常用且强大。所以，需要牢牢地掌握这条命令的使用。

## 使用 Ping 命令

在 Windows 中，ping 命令默认执行 4 次，但是在 Linux 中，只要不使用`ctrl+c`，这条命令就会间隔 1 秒钟，不断执行。如下:
```shell
$ ping www.baidu.com
PING www.a.shifen.com (182.61.200.6): 56 data bytes
64 bytes from 182.61.200.6: icmp_seq=0 ttl=53 time=28.194 ms
64 bytes from 182.61.200.6: icmp_seq=1 ttl=53 time=35.039 ms
64 bytes from 182.61.200.6: icmp_seq=2 ttl=53 time=35.921 ms
64 bytes from 182.61.200.6: icmp_seq=3 ttl=53 time=35.416 ms
64 bytes from 182.61.200.6: icmp_seq=4 ttl=53 time=36.178 ms
64 bytes from 182.61.200.6: icmp_seq=5 ttl=53 time=35.157 ms
64 bytes from 182.61.200.6: icmp_seq=6 ttl=53 time=38.092 ms
64 bytes from 182.61.200.6: icmp_seq=7 ttl=53 time=41.223 ms
64 bytes from 182.61.200.6: icmp_seq=8 ttl=53 time=28.812 ms
```


默认情况下，ping 命令会发送 64 字节的数据给目标服务器。ping 命令是使用 ICMP 协议来发送数据的。icmp_seq 是从 1 开始递增，如果中间递增并不连贯，则表示有丢包的情况发生。

> ICMP 的全称为"Internet Control Message Protocol",中文名为 Internet 控制消息协议。它是TCP/IP协议族的一个子协议，用于在IP主机、路由器之间传递控制消息。控制消息是指网络通不通、主机是否可达、路由是否可用等网络本身的消息。


在截图中`ttl=56`是指经历路由的生存时间，默认是从 64 开始，每当经过一个路由器，则减去 1。

time 是指发送数据包到返回的这个过程所消耗的时间。

最下方，有一个关于 ping 的统计信息：总共发送了 4 个数据包，接收到了 4 个数据包的返回，0 个数据包丢失，总共耗时为 3004 毫秒。rtt 表示的是传输的延时，分别给出了最小值/平均值/最大值以及一个 mdev 值。这个 mdev 值表示 ICMP 包的 RTT 偏离平均值的程度，主要用来衡量网速的稳定性。这个值越大说明网速越不稳定。

下面的表格中，给出 rtt 的参考值:

| 场景         | RTT参考值  |
|------------|---------|
| ping 本机    | 0.01 ms |
| ping 同机房机器 | 0.1 ms  |
| ping 同城机器  | 1 ms    |
| ping 不同城机器 | 20 ms   |
| ping 南北方机器 | 50 ms   |
| ping 国外机器  | 200 ms  |


在不同的操作系统中， mdev 这个参数的名字是不同的，在 mac 系统中，叫做 stddev，在 Windows 中不存在这个参数。

## ping 命令的一些其他参数 {id="args"}

除了默认参数之外，我们还可以指定一些其他的参数。如下示例:

```shell
ping -c 3 www.baidu.com 	## 指定 ping 的次数
ping -s 65500 www.baidu.com		## 指定每次发送的数据包大小
ping -t 255 www.baidu.com ## 指定数据包在网络中的生命时长，超时则被丢弃
ping -i 0.1 www.baidu.com 	## 指定每次发送的时间间隔，小于 0.2 秒则需要 root 用户的权限
ping -f www.baidu.com	## 采用无间隔的方式全力发送探测的数据包，确保每秒钟至少发送 100 个
```
