# 编写基于 CLI 的程序

按照程序的表现形式来划分，分为基于图形化界面的 GUI 程序和基于命令行界面的 CLI 程序。在面向广大电脑用户的程序中，多半都是基于 GUI 的。在面向计算机从业人员的程序中，大部分都是基于 GUI 的。

而对于软件开发者来说，不但要学习如何使用 CLI 程序，而且还要数量掌握 CLI  程序的编写。所以，这篇文档我们就来聊聊如何使用 Python 来编写基于 CLI 的程序。

### 什么是命令行参数

当我们在 Linux 环境下，执行 `ls -al` 的时候，其实就是在运行一个基于 CLI 的程序，这个程序的名称为 `ls` ，在这行命令中，传递了两个参数，分别为 `a` 和 `l` 。其中 `a` 表示罗列所有的所有的文件，哪怕是隐藏的文件。而 `l` 表示以列表的形式展示文件。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/gr3OoHohHk1TnKMJeBva.png" alt="example"/>

这就是命令行参数，大概率你看不到图形界面。如果看到了，那可能像下面这样子:

```python
/**
 *                             _ooOoo_
 *                            o8888888o
 *                            88" . "88
 *                            (| -_- |)
 *                            O\  =  /O
 *                         ____/`---'\____
 *                       .'  \\|     |//  `.
 *                      /  \\|||  :  |||//  \
 *                     /  _||||| -:- |||||-  \
 *                     |   | \\\  -  /// |   |
 *                     | \_|  ''\---/''  |   |
 *                     \  .-\__  `-`  ___/-. /
 *                   ___`. .'  /--.--\  `. . __
 *                ."" '<  `.___\_<|>_/___.'  >'"".
 *               | | :  `- \`.;`\ _ /`;.`/ - ` : | |
 *               \  \ `-.   \_ __\ /__ _/   .-` /  /
 *          ======`-.____`-.___\_____/___.-`____.-'======
 *                             `=---='
 *          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
 *                     佛祖保佑        永无BUG
```

### 接收命令行的参数

在 GUI 程序中，我们可以通过各种图形化的控件和程序进行交互。而在 CLI 程序中，我们可以使用命令行的参数与程序进行交互。那么如何从命令行中获取参数呢？这就需要借助 Python 标准库提供的 `argparse` 模块了。

#### 帮助参数

在 Python 的标准库中，提供了 `argparse` 模块可以用来接受命令行中的参数。下面是一个最简单的示例:

```python
import argparse

parser = argparse.ArgumentParser(description='Demo')
parser.parse_args()
```

运行这个脚本，输出如下:

```shell
$ python3 args.py -h
usage: args.py [-h]

Demo

optional arguments:
  -h, --help  show this help message and exit
```

默认情况下， `argparse` 库会为脚本引入一个 `h` 的参数，显示脚本的帮助信息。如果我们没有添加其他的参数，又在命令行中添加了额外的参数的时候，会出现如下错误的提示:

```shell
$ python3 args.py -ha
usage: args.py [-h]
args.py: error: argument -h/--help: ignored explicit argument 'a'
```

> 注意: 多个参数之间可以不用合并，因此 `-ha` 和 `-h -a` 等效。


#### 添加位置参数

那么如何添加一个参数呢?我们先来演示添加位置参数(除了位置参数外，还有可选参数)。所谓位置参数，就是按照程序中添加参数的顺序，在命令行中书写参数。

在上面代码的基础上，加上如下这句代码，注意要在 `parser.parse_arg()`之前:

```python
parser.add_argument('age', help='Your age')
```

然后运行查看帮助，可以看到，帮助信息中已经存在了这个方法:

```shell
$ python3 args.py --help
usage: args.py [-h] age

Demo

positional arguments:
  age        Your age

optional arguments:
  -h, --help  show this help message and exit
```

#### 获取命令行中传递的参数的值

然后我们需要接受传参的结果，改写一下代码，接受传参:

```python
params = parser.parse_args()
print(params.age + 1)
```

然后执行 `python3 args.py 18` , 结果会报错, 错误如下:

```
Traceback (most recent call last):
  File "args.py", line 6, in <module>
    print(params.age + 1)
TypeError: can only concatenate str (not "int") to str
```

`argparse`不会擅作主张地将你的数值的传参转换为数值，所有的传参都会被当成是字符串类型。你可以自己在代码中强制转换, 例如 `print(int(params.age) + 1)`。

#### 使用可选参数

比如 `ls -al` 中的参数都是可选的，可以不加上参数，也可以颠倒参数的位置为 `ls -la` 。这个如何实现呢？

```python
import argparse

parser = argparse.ArgumentParser(description='Demo')
parser.add_argument('-a', '--age', help='Your age', action='store')
parser.add_argument('-n', '--name', help='Your name', action='store')
params = parser.parse_args()
print(f'My name is {params.name}, {params.age} years old.')
```

然后我们运行这个实例:

```shell
$ python3 args.py -n bob -a 12
My name is bob, 12 years old.

$ python3 args.py --name bob --age 12
My name is bob, 12 years old.
```

可以采用缩写，也可以使用全程。一般缩写使用单中划线，而全称使用双中划线。 `store` 值是 `action` 参数的默认值，表示其是命令行参数的值是字符串。如果 `action` 设置为 `store_true` 会将其转为数组、如果设置为 `append` 会将存储为列表。

> 从 Python3.8 开始，action 的值也可以是自定义的类，自定义处理。



### 参考资料

1. [Argparse 教程 - Python3.8.6rc1](https://docs.python.org/zh-cn/3/howto/argparse.html#id1)
2. [argparse --- 命令行、参数和子命令解析器](