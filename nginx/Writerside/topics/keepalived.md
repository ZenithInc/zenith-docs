# Keepalived 实现高可用

我们可以使用 Nginx 的负载均衡实现集群，但是单节点的 Nginx 本身可能发生单点故障。所以这篇文档介绍了如何使用 Keepalived 实现 Nginx 主备高可用。

## 实验环境说明 {id="remark"}

需要注意的是，环境不可以使用云环境，大概率是行不通的。我使用的是虚拟机环境。实验使用的 Keepalived 版本是 V2.2.7。

另外在开始之前，需要将服务器的防火墙全部关闭:
```shell
systemctl stop firewalld
```
## Keepalived 安装 {id="install"}

进入[官网下载](https://www.keepalived.org/download.html)最新版本即可:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/hbr0ckuu6nJVB4TbBNNZ.png)

下载命令:
```shell
wget https://www.keepalived.org/software/keepalived-2.2.7.tar.gz
```
接着解压:
```shell
tar -zxvf keepalived-2.2.7.tar.gz && cd keepalived-2.2.7
```
需要安装一些依赖:
```shell
yum install gcc openssl-devel libnl libnl-devel -y
```
然后编译安装:
```shell
./configure --prefix=/usr/local/keepalived --sysconf=/etc
make && sudo make install
```
最后创建配置文件:
```shell
cd /etc/keepalived
cp keepalived.conf.sample keepalived.conf
cp /root/keepalived-2.2.7/keepalived/etc/init.d/keepalived /etc/init.d/
cp /root/keepalived-2.2.7/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
systemctl daemon-reload
systemctl enable keepalived
```

## 核心配置文件 {id="configure"}

我们来看看这个配置文件中的内容，最开始`global_defs`中是全局配置，内容如下:

```nginx
global_defs {
  notification_email {				# 主节点发生故障后通知的邮箱地址
    acassen@firewall.loc
      failover@firewall.loc
      sysadmin@firewall.loc
  }
  notification_email_from Alexandre.Cassen@firewall.loc
    smtp_server 192.168.200.1		# 邮件服务器相关配置
    smtp_connect_timeout 30
    router_id LVS_DEVEL        # 当前节点的路由 ID，必须是全局唯一的
  	...省略其他配置...
}
```

再接下来是节点的配置:

```nginx
vrrp_instance VI_1 {
  state MASTER							# 作为 master 节点，如果是备用机则是 BACKUP
    interface eth0          # 当前实例绑定的网卡
    virtual_router_id 51    # 主备节点需要一致，都使用 51
    priority 100						# 优先级权重，优先级高在主节点宕机后成为 MASTER
    advert_int 1            # 主备之间同步检查的时间间隔，默认1秒
    authentication {			  # 认证授权的密码，防止非法节点的进入
    	auth_type PASS
      auth_pass 1111
  	}
    virtual_ipaddress {     # 虚拟 IP 配置
      192.168.1.161
    }
}
```

## 启动 Keepalived {id="started"}

在完成配置之后，我们就可以在主节点上启动它了:

```shell
systemctl start keepalived
```

启动之后查看 IP，多出了我们配置文件中的 VIP:

```shell
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:a7:fc:d8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.115/24 brd 192.168.0.255 scope global noprefixroute dynamic enp0s3
       valid_lft 2960sec preferred_lft 2960sec
    inet 192.168.0.210/32 scope global enp0s3		# 配置文件中的虚拟 IP
       valid_lft forever preferred_lft forever
```

安装 Nginx, 然后启动服务，就可以使用 `192.168.0.115` 或是 `192.168.0.210` 访问我们的 Web 服务。

## 在备用机上启用 Keepalived {id="start-backup-machine"}

按照前文的描述，安装 keepalived, 然后配置好，并启动。启动之后，然后可以将主节点关闭:
```shell
systemctl stop keepalived
```
然后我们来访问虚拟 IP, 可以观察到已经切换到了备用节点:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/ldTyAUFVcDNsPiBnbY3H.png)

恢复主节点之后，就会重新切换到主节点:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/Ut0bLsLPrlWHdqcsjZ7Z.png)

在主备节点切换的过程中，我们 `192.168.0.210` 这个 VIP 也会在主备间切换绑定。

## 添加检测脚本不间断服务 {id="script"}

上面我们已经演示了，如果 Keepalived 宕机，则会切换到备用节点。但是如果只是 Nginx 宕机呢？则会发生服务不可用，不会主动切换到备用节点。

所以我们可以写一个脚本来检测 Nginx 是否宕机，如果宕机，则尝试重启，如果重启失败，则直接停止 Keepalived 然后就会切换到备用节点。

```shell
#!/bin/bash

A=`ps -C nginx --no-header |wc -l`
# 判断nginx是否宕机，如果宕机了，尝试重启
if [ $A -eq 0 ];then
    systemctl start nginx
    # 等待一小会再次检查nginx，如果没有启动成功，则停止keepalived，使其启动备用机
    sleep 3
    if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
        killall keepalived
    fi
fi
```
需要为这个脚本赋予执行权限:
```shell
chmod +x check_nginx_alive_or_not.sh
```
接着，我们来修改 `keepalived.conf`来实现自动执行脚本:
```shell
vrrp_script check_nginx_alive {
    script "/etc/keepalived/check_nginx_alive_or_not.sh"
    interval 2 # 每隔两秒运行上一行脚本
    weight 10 # 如果脚本运行成功，则升级权重+10
    # weight -10 # 如果脚本运行失败，则升级权重-10
}
```
然后修改实例配置：
```shell
vrrp_instance VI_1 {
    # ...省略其他配置...
    track_script {
        check_nginx_alive   # 追踪 nginx 脚本
    }
}
```
接着，我们可以使用`systemctl stop nginx`停止 nginx 服务来观察网页是否显示正常。

## 实现双主热备 {id="two-master-hot-backup"}

在之前的方案中，存在一个资源浪费的问题。如果 master 节点非常稳定，那么 backup 节点就会出现常年闲置的情况。于是我们就有了双主热备的方案。如下图所示:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/8eylhaFUwUiqT9FPoZbP.jpeg)

DNS 轮询在域名服务商处配置多条 A 解析，然后配置负载均衡权重未“均等负载”，设置对应权重。

然后在主备两台服务器上各自多配置一组 `vrrp_instance`，原来的 master 新增 backup 实例，而 backup 新增 master 实例即可。示例配置如下:
```shell
vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.161
    }
}

vrrp_instance VI_2 {
    state BACKUP
    interface ens33
    virtual_router_id 52
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.162
    }
}
```