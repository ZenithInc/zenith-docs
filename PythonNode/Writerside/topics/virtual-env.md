# 使用虚拟环境

为什么需要使用虚拟环境呢？因为在不同的项目中，我们使用的环境会有差别。比如 A 项目使用 Flask1.0 版本，B 项目使用 Flask1.1 版本。在不使用虚拟环境的情况下，两个版本就不能共存。这时候，你总不能使用两台机器来开发两个项目吧？

虚拟环境在 Python 的世界里应运而生，但这也成了大部分学习它的一道门槛。总的来说，有两个问题，不知为何物，不知有何用？

在 Python3 之前，社区提供了好多方案，这有时候会让人难以选择。在 Python3 之后，官方提供了 venv 模块，可以方便的使用。

### 为什么不使用虚拟机

我不知道有没有人和我有一样的困惑，但是官方对此是有说明的:

> The [`venv`](https://docs.python.org/3/library/venv.html#module-venv) module provides support for creating lightweight “virtual environments” with their own site directories, optionally isolated from system site directories.


关键词是 lightweight，意思是轻量级的。这大概也是 Python 社区选择这类解决方案的原因。但是随着 Docker 等虚拟化技术的发展、普及，我觉得这类基于系统目录的虚拟环境方案会被取代。

### 创建虚拟环境

首先我们需要创建一个工作目录, 用于存放很多的项目:

```shell
mkdir workspace && cd workspace
```

然后我们创建一个项目，名字叫做 venv_test：

```shell
mkdir venv_test && cd venv_test
```

然后创建虚拟环境:

```shell
python -m venv .
```

venv 是 python 官方提供的一个模块，要调用这个模块，需要使用 python -m。或者你也可以将 venv 理解成 python 内置的一条命令，它支持一个参数，指定创建虚拟环境的目录，在上面的示例中， `.` 表示的是当前目录。就是在当前目录创建一个虚拟环境。

### 激活虚拟环境

接着，我们需要激活虚拟环境，使用如下命令:

```shell
source ./bin/activate
```

执行这条命令之后，我们可以看到在命令行在最前面出现了 `(venv_test)` 的提示，这个是我们的目录名称，提示我们已经进入了虚拟环境:

```
(venv_test) [dc2-user@10-255-0-106 venv_test]$
```

接下来，我们所有的操作都是在虚拟环境里操作的，不影响虚拟环境之外的内容。

### 在虚拟环境中安装依赖包

然后我们就可以在虚拟环境中安装依赖包了, 比如 flask 框架:

```shell
pip install flask
```

比如我们可以升级 pip 的版本:

```shell
pip install --upgrade pip
```

这时候，我们查看一下虚拟环境中的包依赖:

```shell
$ pip list
Package      Version
------------ -------
click        7.1.2
Flask        1.1.2
itsdangerous 1.1.0
Jinja2       2.11.2
MarkupSafe   1.1.1
pip          20.1.1
setuptools   39.2.0
Werkzeug     1.0.1
```

### 退出虚拟环境

如果要退出虚拟环境，我们可以使用 `deactivate` 。