# 快速入门

当你学习了 Python 的基本语法之后，是不是有这样的困惑？我好像懂了，但是我不知道我能干什么？不干点什么的话，是不是很快就忘了？

是的，应该做点什么，那做什么呢？比较通用的一个做法就是写一个网站。那么问题又来了，网站？不知道从和入手？我说，应该从一个 Web 框架入手。我推荐使用 Flask, 因为它简洁的风格。

### 什么是框架

什么是框架？用最简单的话来说，本质上就是别人写好的代码，因为写得好，因为完成了很多功能，所以提供出来给其他人用。

什么是 Web 框架？也是别人写好的代码啊，包含一些 Web 中常用的功能，这样别人写的时候就可以基于这些写好的功能开发具体的业务了。

所以说，**框架就是别人写好的、写得好的代码，用别人的写好的框架可以加速我们的开发速度**，让我们专注于业务逻辑的开发。

### 框架安装

Python 推荐使用当前最新的稳定版，不应该在使用 Python2.x 的版本了。在安装了 Python3 之后，创建一个虚拟环境:

```shell
mkdir flask_test && cd flask_test
python -m venv . && source ./bin/activate
```

然后安装 flask 并且更新 pip：

```shell
pip install flask
pip install --upgrade pip
# 查看flask版本
python -m flask --version
```

### Hello World

我们创建一个代码文件，命令为 `app.py` , 写入如下内容:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

然后，运行我们的程序:

```shell
export FLASK_APP=app.py;flask run --host=0.0.0.0
```

如果屏幕输出以下文案则表示运行成功:

```
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

最后，我们访问 http://127.0.0.1:5000 就可以看到 hello world 的提示了。如果是服务器环境，则将 127.0.0.1 替换成自己的 ip 。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/kpKwPJiBtAH10lOqyyxa.png" alt="hello-world"/>

### 设置为开发环境

如果我们在浏览器中，访问 `http://127.0.0.1:5000/not_found_page`, 然后会发现页面输出类似效果:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/3QXYaCKTPhASg3q4ajjA.png" alt="not_found"/>

这是因为没有设置为开发环境。设置为开发环境之后，Flask 会开启调试模式，展示更多的错误信息在页面上，以及可以热加载我们的代码文件，当文件内容发生变化之后，自动加载代码并更新页面。如何设置为开发环境呢？设置一个环境变量就好了:

```shell
export FLASK_ENV=development
```

> 如果是在生产环境，不能设置为开发环境，否则因为暴露了更多的错误信息，会有更多的安全隐患。

将 `FLASK_ENV` 设置成 development 将会自动开启调试模式，也可以单独关闭调试模式:

```shell
export FLASK_DEBUG=0
```

> 注意: 如果不关闭调试模式，会导致 PyCharm 等 IDE 的断点调试无效。

### 使用路由

**什么是路由呢？你可以理解成是我们接口的名字就可以了，每一个接口都有一个唯一的名字** 。当我们访问 `http://ip:5000/<接口的名字/路由>`  就可以访问到对应的接口了。

在上面的例子中，我们已经使用了路由了:

```python
@app.route('/')
def hello():
    return 'Hello World'
```

`@app.route('/')` 就是在 Flask 中给接口(这里的接口实际上就是 hello 方法)取一个名字，这样用户才可以访问的到这个接口，取得名字就叫做 `/` 。我们也可以改为 `@app.route('/hello')` ，然后重新启动 flask 服务看看？

```python
@app.route('/hello')
def hello():
    return 'Hello World'
```

这时候，我们要访问就需要通过 `http://ip:5000/hello` 来访问接口了。所以， **大部分情况下，接口和路由是一对一的关系** 。

然后，我们添加一个接口:

```python
@app.route('/user')
def user()
    return 'My nickname is admin'
```

然后访问该接口: `http://ip:5000/user` ，然后你可以在页面上，看到 My nickname is admin 的输出文本。

### 渲染模板

上面的例子，我们都是简单的返回一个字符串。那么如果我想返回一个 HTML 文件呢？就需要采用渲染模板这个功能呢？什么是模板？什么是渲染？是不是有点难以理解？没关系，这两个问题先放一放。 **我们先把渲染模板理解成返回 HTML 文件就可以了** 。

Flask 和我们约定，模板应该放在代码根目录下的 `templates` 文件夹中, 我们在这个文件夹中创建一个 index.html 文件，并写入以下内容:

```html
<h1>Hello</h1>
```

然后，我们修改以下 `app.py` 文件:

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')
```

Flask 提供了 render_template 函数，可以将我们的 html 文件返回给浏览器（这句话并不严谨，暂且这么理解)。
A

### 接收传参

在实际的生产过程中，我们大部分的数据都来源于用户。那么我们如何接收用户通过前端传来的参数呢？对此，Flask 程序提供了 Request 对象，通过它我们就可以获取前端的传参:

```python
from flask import request

@app.route('/login')
def login():
    print(request.from['username'] + '/' + request.from['password'])
    return 'success'
```

在移动化大行其道的今天，移动设备更倾向于使用 `application/json`  格式向服务端传送数据，如何接受这样的数据呢, 使用 `request.json` 即可。

### 返回响应

首先，我们定义一个视图函数:

```python
@app.route('/')
def index():
    return 'Hello World'
```

在这个视图函数中，我们返回了一个字符串。通过浏览器以及路由，我们可以顺利看到这个字符串出现在页面上。

如果我们将 `Hello World` 字符串更换成 `<html></html>` 呢？你将什么也看不到？

```python
@app.route('/')
def index():
    return '<html></html>'
```

为什么呢?因为浏览器会将这串字符串解析成 HTML 网页，在 HTML 网页中，HTML 标签是不显示的。那么浏览器为什么要将这字符串当作 HTML 网页来处理呢？是 Flask 告诉它的:

当 Flask 看到你返回了一个 `<html></html>` 字符串的时候，会检测这个字符串中是否包含 HTML 的标签，如果包含则将返回给浏览器的响应头 (Response Header） 中的 `Content-Type` 的值设置为 `text/html; charset=utf8` 。

这时候浏览器收到了服务器返回的响应，会首先检查返回内容的类型，即 `Content-Type` , 看到它的值包含`text/html` ，于是就当作了 HTML 网页解析。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/FM5KfirAFxKykskpWBUQ.png" alt="dev_tools"/>

如果我希望 `<html></html>` 就是当作字符串输出，而不是解析成 HTML 网页，应该怎么做呢？修改 Flask 的 Response 对象。为此，我们应该先通过 Flask 提供的函数 `make_response()` 来获取 Response 对象:

```python
from flask import Flask, make_response
app = Flask(__name__)

@app.route('/')
def hello():
    response = make_response('<html></html>', 200)
    response.headers['Content-Type'] = 'text/plain'
    
    return response
```

然后我们访问网页，就可以看到浏览器按照字符串输出，而非 HTML 网页输出了:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/kg6i946vd4CxspqiJLi6.png" alt="html"/>

通过这个例子，我们可以了解到 Flask 会通过构造 Response 对象的方式，将我们返回的内容封装成符合 HTTP 协议的文本格式(包含内容) 返回给浏览器。

我们可以通过 Response 对象来修改返回的具体内容，比如说 Content-Type, 从而达到让浏览器以我们想要的方式展示返回的数据。

下面展示一种更为简洁的写法:

```python
import json
from flask import Flask
app = Flask(__name__)
@app.route('/')
def index():
	response = {"name": "JinZhiSu", "email": "admin@end.wiki"}
	return json.dumps(response), 200, {"content-type": "application/json"}
```

### 附录

#### 端口占用问题

ip 是每一个网络上的主机的唯一标识，就像是门牌号。但是啊，一栋房子里可能住着好几户人家，所以，还划分有房间号。

每一个联网的程序(需要接收信件的房间) , 都需要首先向操作系统(房子的主人)申请一个端口(房间号)。

那么，可不可以两个房间申请一个房间号呢？肯定是不可以的，因为这样一封信来了，就不知道给哪个房间了。

所以，在一台主机上，不可能会有两个程序占用一个端口的情况。如果出现这种情况，操作系统就会提示: Address already use， 说明地址已经被使用。这里的地址，指的就是端口号。

所以，如果出现端口已经被占用，比如说 5000 端口，那么需要查找到端口被哪个程序占用了:

```shell
$ sudo lsof -i:5000
COMMAND   PID     USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
flask   22432 dc2-user    3u  IPv4 8669596      0t0  TCP *:commplex-main (LISTEN)
```

当中会显示被 flask 占用了，其进程 ID  为 22432。最后结束这个进程:

```shell
$ sudo kill -9 22432
```

结束之后， 5000 端口也就被释放了，其他的程序就可以重新向操作系统申请这个端口了。所以，可以重新运行 `flask run`了。