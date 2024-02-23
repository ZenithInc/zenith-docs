# Hello World

本文就 python 开发环境的安装配置做一些说明。

## Hello World

使用 Python 最快的方式就是通过在线的 Playground 网站，比如 [programiz.pro](https://programiz.pro/ide/python)。下面是一个 Hello World 的示例:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/KcMSCl8lVNiXCqvG0Bcg.jpeg" alt="playground online"/>

这也是我最为推荐的方式，对于新手来说避免了编程环境这只巨大的拦路虎。

## Python 版本的选择

Python 采用了两段式的版本号 `<Main>.<Subject>` , 比如当前最新版本是 3.8。大的版本号分为 2.X 和 3.X。没有例外，应该全部选择 3.X 版本，2.X 已经走入了历史。

在 3.X 的众多版本中，其增加了如下的特性:

### Python 3.0

- 函数注解（ Function Annotations ），可以对函数的参数和返回值进行注解。除了可以通过 `__annotations__` 来访问这些注解外，没有其他更多的语义；
- 仅限关键字形参（ Keyword-only arguments ），在函数形参中使用 `*` 来表示对应的形参必须使用关键字进行传递；
- `nonlocal` 语句允许嵌套函数中的内部函数对外层函数作用域中的变量进行赋值；
- 扩展的 Iterable 解包，比如 `(a, *rest, b) = range(5)；`；
- 集合字面量，字典推导，新的八进制字面量，以 `b` 或 `B` 开头的字节串等等；

### Python 3.1

- 有序字典 `collections.OrderedDict`；
- 千位分隔符的格式说明符，比如 `'{0:,d}'.format(1234567)` —> `'1,234,567'`；

### Python 3.2

- Argparse 命令行解析模块；
- 基于字典的日志配置 `logging.config.dictConfig()` 可以支持更灵活的配置方式；
- concurrent.futures 模块 ；
- 将字节码文件单独存储在 `__pycache__` 目录；

### Python 3.3

- 虚拟环境工具 venv 和 pyvenv，后者 pyvenv 在 Python 3.6 中已弃用；
- 隐式命名空间包；
- OS 和 IO 异常的层次结构优化；
- 委托给子生成器 `yield from`；
- 允许禁用链式异常上下文的显示从而获取更干净的错误消息，比如 `raise AttributeError(attr) from None`；

### Python 3.4

- pip 默认包含在 Python 的二进制安装程序中；
- 异步 I/O 库 asyncio；
- 支持枚举的模块 enum；
- 面向对象的文件系统路径模块 pathlib；
- 高级 I/O 复用模块 selectors；
- 用于数学统计的模块 statistics；

### Python 3.5

- 使用 async 和 await 语法实现原生协程 ；
- 矩阵乘法运算符 `a @ b` ；
- 类型标注支持的模块 `typing`；
- 进一步扩展带有 `*` 的可迭代对象解包操作和带有 `**` 的字典解包操作；

### Python 3.6

- 文字字符串插值，比如 `f"He said his name is {name}."`；
- 用于进一步增强类型标注的变量注释语法，比如 `names: List[str] = []`；
- 异步生成器，允许在 `async` / `await` 中使用 `yield`；
- 使用 `async for` 进行异步推导；
- 用于生成高加密强度的随机数、密码、安全凭证的模块 `secrets`；

### Python 3.7

- 延迟的类型标注求值；
- 用于 Python 调试的内置 `breakpoint()` 函数；
- 用于支持上下文变量的 `contextvars` 模块；
- 声明数据类的 `dataclass()`，比如：

```python
@dataclass
class Person:
    name: str
    height: float
    weight: float = 130.0
p = Point('Bob', 175.5)
print(p)   # 输出 "Person(name=Bob, y=175.5, z=130.0)"
```

### Python 3.8

- 赋值表达式语法 `:=`；
- 仅限位置形参（ Positional-only parameters ），在函数形参中使用 `/` 来表示对应的形参必须使用位置进行传递。

另外就是海象运算符( := )。我们先来说说，如果没有它，我们的代码怎么写:
```Python
str = 'Hello World'
str_len = len(str)
if str_len > 5:
    print('to do somethings use str_len: {}'.format(str_len))
```
如果使用了海象运算符，代码又该怎么写？
```Python
str = 'Hello World'
if (str_len := len(str)) > 5:
    print('to do somethings use str_len: {}'.format(str_len))
```
<note>
注意: 因为海象运算符的运算级别比 > 这样关系运算符要低，所以需要加上 () 来保证其先赋值。
</note>

所以， 用于不用海象运算符有什么区别？答案很明显，少了一行代码。提供了一种语法糖，让代码更加的简洁、更具表达力。

## Pip 的使用

pip 是 Python 自带的一个包管理的工具，可以使用对项目中的依赖进行管理。就其基础的使用，如下实例:

```shell
pip install <package> [package...]    ## 安装一个多多个包
pip install <package>==2.21.0         ## 安装指定版本的包
pip install --upgrade <package>       ## 升级指定的包
pip uninstall <package>               ## 卸载指定的包
pip list                              ## 打印已经安装的包的列表
pip freeze > requirements.txt         ## 生成已经安装的软件包的列表，并重定向到 requirements.txt 文件中
pip install -r requirements.txt       ## 根据 requirements.txt 文件中的内容安装项目所需依赖
```

如果在使用 pip 的过程中出现安装超时的现象，可能是因为国外服务器在国内不稳定的缘故，可以换成豆瓣的镜像服务器下载资源，这样会快很多。编辑配置文件 `~/.pip/pip.conf` ， 内容如下:

```
[global]
index-url = https://pypi.doubanio.com/simple
trusted-host = pypi.doubanio.com
```

## Python 编码规范 {id="style-for-python-code"}

Python 目前主要有两份编码规范, 开源社区遵循的 [PEP8](https://peps.python.org/pep-0008/) 和 [Google Python](https://google.github.io/styleguide/pyguide.html) 编码规范。

我觉得编写 Python，不管是个人还是团队，没有必要自行创造一份规范，这两种中选择一种即可。

## Jupyter Notebooks

首先安装 Anaconda， 在其[官方页面](https://www.anaconda.com/products/individual)上下载安装包安装即可。安装的过程中，建议勾选，将可执行文件加入 PATH 的选项。

在 Anaconda 中已经包含了 Jupyter Notebooks ，直接在命令行下运行即可:

```shell
jupyter notebook
```

## 一些参考资料以及推荐

1. [《深入理解Python特性》](https://book.douban.com/subject/34262228/)