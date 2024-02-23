# 使用 GoAccess 监控 Nginx

使用 GoAccess 这款软件可以非常方便的监控 Nginx 的 Access 日志，并以非常可观的界面展示在 Web 页面中。

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/BQP0d3obfVmGywEkuHBm.jpeg)

## 安装 GoAccess {id="install"}

需要安装 `ncursesw` 这个依赖:
```shell
sudo yum install ncurses-devel ncurses geoip-devel -y
```
然后，安装 GoAccess:
```shell
wget http://tar.goaccess.io/goaccess-1.2.tar.gz
tar -xzvf goaccess-1.2.tar.gz
cd goaccess-1.2/
./configure --enable-utf8 --enable-geoip=legacy
make && sudo make install
sudo ln -s /usr/local/bin/goaccess /usr/bin/goaccess ## 建立软链
```

## 基础使用 {id="usage"}

启用实时的监控如下:
```shell
goaccess /var/log/nginx/access.log 
	-o /usr/share/nginx/html/report.html 
  --real-time-html --time-format='%H:%M:%s' 
  --date-format='%d/%b/%Y' 
  --log-format=COMBINED 
  --port=5260 
  --addr=0.0.0.0
```

具体的使用可以参考 [官方的文档](https://www.goaccess.cc/?mod=man)。