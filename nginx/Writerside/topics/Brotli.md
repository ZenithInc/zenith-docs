# 使用 Brotli 压缩

今天晚上，突然我们公司的服务全面挂了。经过排查，发现是因为服务的带宽超过了阿里云的 300MB 限制。那么是什么原因导致的突然带宽猛涨呢？是因为产品
发了一条微博，吸引了大量的流量。问题发生了如何应对呢？首先是联系阿里云，将带宽的限制由 300MB 提升到了 1GB。另外，网关启用 Brotli 压缩算法。
所以，这篇文档我们就来说说 Brotli 压缩算法在 Nginx 上的应用。

## Brotli 压缩算法 {id="brotli"}

根据维基百科的描述，是 Google 的员工 Jyrki Alakuijala 和 Zoltan Szabadka 最初开发了 Brotli 算法，初衷是为了降低 WOFF 格式的字体在
Web 的传输过程中的体积。在 2013 年的时候。直到 2016 年，他们又完成了 Brotli 规范。Brotli 是一种新的数据格式，进一步提供了压缩比。Brotli 
规范在 2015 年 9 月被推广用于 HTTP 流压缩。

截止 2022 年 7 月，大部分的浏览器已经支持 Brotli 规范，超过 96% 的互联网用户的浏览器是支持的。所以，可以放心的使用。

## 安装 {id="install"} 

下载 Nginx 源码, 解压:
```Shell
$ wget wget https://nginx.org/download/nginx-1.24.0.tar.gz
$ tar -zxvf nginx-1.24.0.tar.gz
```
接着，进入 nginx 源码的根目录，并克隆 Brotli 的项目源码:
```Shell
$ cd nginx-1.24.0
$ git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
$ cd ngx_brotli/deps/brotli
$ mkdir out && cd out
$ cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..
$ cmake --build . --config Release --target brotlienc
```

然后，回到 nginx 的源码目录下:
```Shell
$ ./configure \
--add-module=ngx_brotli \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_gzip_static_module
```

如果缺少依赖:
```Shell
$ sudo dnf install pcre pcre-devel -y
```

然后编译安装:
```Shell
$ make -j4 && sudo make install
```

## 配置 {id="configure"}

配置 Brotli 压缩如下:
```NGINX
http {
    brotli on;
    brotli_static on;
    brotli_types text/plain text/css application/javascript application/json image/svg+xml application/xml+rss;
}
```

然后重启即可。

## 测试 {id="test"}

最后我们简单测试一下，生成一个 20MB 的文本文件:
```Shell
$ sudo sh -c "head -c 20M </dev/urandom | base64 > random_text.txt"
$ ll -h random_text.txt
-rw-r--r-- 1 root root 28M Jan 19 22:03 random_text.txt
```

> 需要注意的是，在浏览器中请求这个文件，要看一下请求头中的 `Accept-Encoding:gzip, deflate, br`是否包含 `br`, 我在 Edge 浏览器中发现默认并
没有包含 `br`, Safari 浏览器也可能不支持。

然后观察文件传输的大小，以及响应头中的 `Content-Encoding` 字段是否为 `br`:

![response](http://file-linker.oss-cn-hangzhou.aliyuncs.com/JsgbyDNGdOsmyYvPYYH4.jpeg)

## 总结 {id="summary"}

本文详细介绍了 Brotli 压缩算法的背景、发展和应用。Brotli 是由谷歌员工开发的，旨在降低网络传输中的数据体积。截至 2021 年 3 月，Brotli 
已得到大部分浏览器的支持。

接着进一步提供了详细的 Nginx 上 Brotli 安装和配置步骤，包括下载和编译 Nginx 源码、配置 Brotli 模块、解决依赖问题和编译安装。配置部分详细
说明了如何在 Nginx 中启用 Brotli 压缩。

最后，通过生成一个 20MB 的文本文件来测试 Brotli 压缩的效果。测试中要注意检查浏览器请求头中是否包含 Brotli 编码，并观察文件传输大小及
响应头中的 Content-Encoding 字段。