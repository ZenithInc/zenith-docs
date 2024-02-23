# 使用 stub_status 监控 Nginx

Nginx 的 stub_status 模块提供了关于 Nginx 服务器状态的基本信息，这对于监控和分析服务器性能非常有用。

## 示例 {id="usage"}

首先要确保已经启用了 `stub_status` 模块:

```Shell
# 查看 nginx 的编译信息
# 输出中应该包含 --with-http_stub_status_module
nginx -V
```

配置 nginx:
```NGINX
server {
    listen 8000;
    server_name _;

    location /status {
        stub_status on;
        allow 192.168.0.1; 
        allow 192.168.0.2;
        deny all;
    }
}
```
在上面的配置中，我们启用了 `stub_status`，并且允许 `192.168.0.1` 以及 `192.168.0.2` 两个内网 IP 可以访问监控信息，并且 `deny` 其他
IP 的访问。

当我们通过访问 `http://192.168.0.1/status` 访问监控，页面呈现内容如下:
```Text
server accepts handled requests
 823007 823007 310893 
Reading: 0 Writing: 1 Waiting: 0
```

Nginx 的 `stub_status` 页面显示了一些关于服务器性能和连接状态的关键信息。下面是对你提供的输出内容的解释：

1. **连接和请求统计：**
    - `823007`: 这个数字表示从服务器启动到现在，Nginx 接受（accepted）的连接总数。
    - `823007`: 这个第二个数字表示Nginx 已经处理（handled）的连接总数。通常这个数字和接受的连接总数相同，但如果出现某些连接处理失败的情况，
这两个数字可能会有所不同。
    - `310893`: 这个数字代表服务器从启动开始处理的总请求数。这包括所有的 HTTP/HTTPS 请求。

2. **当前连接状态：**
    - `Reading: 0`: 服务器当前正在读取客户端请求头的连接数为 0。
    - `Writing: 1`: 服务器当前正在向客户端发送响应的连接数为 1。
    - `Waiting: 0`: 当前处于等待状态的连接数为 0。等待状态通常指的是已经完成读取请求和发送响应，正在等待进一步动作的连接。

这些数据对于理解服务器的当前负载和性能非常有用。例如，如果你看到大量的“等待”（waiting）连接，这可能是一个潜在的性能瓶颈的迹象。同样，如果处理
的连接数远小于接受的连接数，这可能表明有连接错误或配置问题。

## 自动化 {id="automation"}

对于持续监控和分析，你可能希望将 stub_status 与监控工具（如 Prometheus、Grafana 或 Zabbix）集成。这通常涉及到安装和配置额外的模块或代理，
以定期收集并可视化这些数据。