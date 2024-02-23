# OpenResty

这篇文档介绍了 OpenResty 相关的基础知识，从安装到入门。更加详细的资料可以参考官方文档。

## 简介 {id="intro"}

OpenResty 是一个内嵌 Nginx 和 LuaJIT 的 Web 平台。包含了大量高质量的 Lua 库以及第三方依赖。你可以使用它作为 Web 网关，甚至可以编写动态的 Web 应用。它拥有超高的性能、可以通过异步 I/O 和数据库、缓存进行交互。

## 安装 {id="install"}

更详细的下载和安装内容可以参考[官方文档](https://openresty.org/en/installation.html)。下文以在 Linux（Rocky 9.2） 上，通过包管理工具安装。

```Shell
# add the repo
wget https://openresty.org/package/rocky/openresty2.repo
sudo mv openresty2.repo /etc/yum.repos.d/openresty.repo

# update the index
sudo yum check-update

# install
sudo yum install -y openresty openresty-resty
```

## Hello World {id="hello-world"}

最简单的方式是使用命令行输出一行“Hello World”字符，这样可以验证是否已经正确安装:

```Shell
resty -e 'print("Hello World")'
```

然后可以尝试通过启动 OpenResty 的服务，新建一个虚拟站点，展示一个 `Hello World` 的页面。这部分可以参考[官方文档](https://openresty.org/en/getting-started.html)。首先创建工作目录:
```Shell
mkdir work/
cd work/
mkdir conf/ logs/ scripts/
```

这部分和配置 Nginx 并无两样，特殊的是，我们并不需要通过创建一个`index.html` 的页面，而是可以直接通过在 Nginx 的配置中嵌入 lua 脚本输出 `Hello World`。Nginx 的配置如下:
```NGINX
server {
    listen 80;
    location / {
        default_type text/html;
        # 使用 LUA 脚本输出 `hello world`
        content_by_lua_block {
            ngx.say("<p>hello, world</p>")
        }
    }
}
```

## 提交表单 {id="submit-form"}

当你完成了上面 `hello world` 的示例后，可以尝试下面这个提交表单的示例继续学习 OpenResty。

首先，创建 Nginx 的配置文件，命名为 `work/conf/nginx.conf`, 内容如下:
```NGINX
location /post {
    default_type 'application/json';
    content_by_lua_file scripts/post.lua;
}
```
示例中，我们通过 `content_by_lua_file` 来指定处理请求的 Lua 脚本文件，其内容如下:

```NGINX
-- post.lua
ngx.req.read_body()
local args, err = ngx.req.get_post_args()

if not args then
    ngx.status = 400
    ngx.say("Bad Request: ", err)
    ngx.exit(ngx.HTTP_BAD_REQUEST)
end 

local post = {
    title = args.title,
    content = args.content
}
```
这个短小的示例中，展示了很多脚本的元素:

* `ngx.req.read_body()` 在获取请求参数之前，需要调用 `ngx.req` 对象的 `read_body()` 方法，读取请求参数。
* `local` 关键字定义变量
* `ngx.req.get_post_args()` 方法可以读取 `post` 的请求参数，并返回两个参数，分别是 `args` 和 `err` 。
* 通过 `if...then...end` 的结构编写分支语句
* `ngx.status` 属性可以更改返回客户端的状态码
* `{}` 定义字典(map)数据结构

## 在页面中输出 JSON {id="response-by-json"}

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/JL3ZD5G7PdwvQ6NwBHHT.png" alt="output json"/>

接着上面这个例子，在 Nginx 配置中加入如下配置:
```NGINX
location /posts {
    default_type 'application/json';
    content_by_lua_file scripts/posts.lua;
} 
```
脚本内容如下:
```NGINX
-- posts.lua
local json = require "cjson"

local posts = {
    { title = "Post 1", content = "Content of post 1" },
    { title = "Post 2", content = "Content of post 2" },
}

ngx.say('{"posts": ', json.encode(posts), '}')
```
我们可以将模块通过 `require` 这个关键字引入，比如`cjson` 可以对数据进行序列化或反序列化。    

OpenResty 内置了很多的常用库, 如下表所示:

| 常用库                | 描述                                          |
|--------------------|---------------------------------------------|
| `resty.core`       | OpenResty 的核心库，包含了一些基本的功能，如字符串处理、正则表达式、表操作等 |
| `resty.cookie`     | 用于处理 HTTP Cookie 的库，方便读取和设置 Cookie          |
| `resty.aes`        | 提供了 AES 加密和解密的功能                            |
| `resty.http`       | 允许进行 HTTP 请求，与外部服务进行通信                      |
| `resty.redis`      | 用于与 Redis 数据库进行交互                           |
| `cjson`            | JSON 编码和解码库，用于处理 JSON 数据                    |
| `lua-resty-jwt`    | 处理 JSON Web Token（JWT）的库，用于身份验证和授权          |
| `lua-resty-mysql`  | 用于与 MySQL 数据库进行交互                           |
| `lua-resty-upload` | 于处理文件上传                                     |

以 `aes` 解密举例:
```NGINX
local aes = require "resty.aes"

local ui = "lEpl%2BBt4U2lgKDJnnehI2ynsG4B%2FNcP%2FVK7fmYXdw7QfC5y2CwAZ%2BZN21QIpQ6kzdSk%2BLH2gYsULveGpdRF7e59b2wgcYQ74F1lmU84EF%2FA%3D"
local decoded = ngx.decode_base64(ngx.unescape_uri(ui))
local decodedLength = string.len(decoded)
local iv = string.sub(decoded, -16, decodedLength)
local aes_256_cbc_md5 = aes:new("your password", nil, aes.cipher(256, "cbc"), {iv=iv})
local decrypted = aes_256_cbc_md5:decrypt(decoded)
```

## OPM-包管理工具 {id="opm"}

OPM 是官方的包管理工具，安装如下:
```Shell
sudo dnf install openresty-opm -y
```

下面的命令演示如何安装依赖，以 `ktalebian/lua-resty-cookie` 为例，这是一个处理 Cookie 的包，由 `CloudFlare`的工程师开发:
```Shell
opm get ktalebian/lua-resty-cookie
```

## Cookie {id="cookie"}

上文已经介绍了如何使用 OPM 安装依赖，例如 `ktalebian/lua-resty-cookie`, 接下来介绍如何在操作 Cookie。例如获取 Cookie:

```NGINX
local ck, err = cookie:new()
if not ck then
    ngx.say(ngx.ERR, err)
end

local ns, err = ck:get("ns")
```

设置 Cookie 示例如下:

```NGINX
local function set_cookie(name, value, max_age)
    local cookie = require "resty.cookie"
    local ck = cookie:new()
    local ok, err = ck:set({
        key = name,
        value = value,
        path = "/",
        max_age = max_age,
        secure = false,
        httponly = true
    })

    if not ok then
        ngx.log(ngx.ERR, "Failed to set cookie: ", err)
        return false
    end
    return true
end
```

清除 Cookie:
```NGINX
local function clear_cookie(name)
    local expires = "Expires=Thu, 01 Jan 1970 00:00:01 GMT;"
    local path = "Path=/;"
    local cookie = string.format("%s=; %s %s", name, expires, path)
    ngx.header["Set-Cookie"] = cookie
end
```


## 参考资料 {id="also-articles"}

1. [官方文档](https://openresty.org)
2. ChatGPT