# 命令是如何执行的 

在开始学习很多很多的命令之前，我们先来说说一条命令是如何执行的。只有知道了“命令”的本质，我们才能更快更深入的去学习更多的内容。

## 命令的本质 {id="what-is-command"}

我们以`date`命令为例，这条命令可以打印系统时间:

```shell
$ date
2021年 4月20日 星期二 15时29分46秒 CST
```

然后我们再介绍一条有用的命令，`which` ,这条命令可以找到`date`命令背后的程序所在的位置:

```shell
$ which date
/bin/date
```

最后，再来介绍一个命令，`file`, 我们可以使用这条命令来返回一个文件的类型:

```shell
$ file /bin/date
/bin/date: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=c2b6122cb2c8789f66de3342471f9d36b3f722ad, stripped, too many notes (256)
```

哦，原来**命令的本质就是一个别人写好了软件啊，换句话说就是一个可执行文件**啊。何以见得呢？你看输出信息中有 ELF 64-bit 字样，这是什么意思？可执行文件嘛。

## 命令在哪里 {id="where-is-commands"}

在上文中，我们已经知道了，`date`命令在`/bin/`目录下。那么其他的命令呢？

```shell
$ which cd
/usr/bin/cd
```

嗯嗯，和`date`命令是存放在不同的目录下的，唯一的相同点是都在一个名为`bin`的子目录下，bin 是  binary 的缩写。不管是在 Linux 下或者 Windows 下都习惯把可执行文件存放在`bin`目录下，表示它是二进制的可执行文件。

问题来了，既然这些命令都存放在不同的目录下，当我们在命令行中输入一条命令的时候，Linux 怎么知道它是在哪一个目录下呢？换一种问法：Linux 是如何找到我们在命令行中输入的命令对应的可执行文件的？

基于约定，Linux(其实是一个名为 Bash  的软件是用来解释命令的) 和我们约定在环境变量`path`中定义的目录中存放可执行文件。或者说，当我们在命令行下输入一条命令的时候，它会去`path`变量中定义的目录中去一个一个的找，找到了就执行，找不到就提示`Command not found!`:

```shell
$ echo $PATH
/root/.config/composer/vendor/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```

也就是说在 Linux 下，你只要把你的可执行文件存放在上面使用冒号分隔的任意一个目录中，都能在命令行中输入命令的时候找到对应的程序并执行。那么`path`这个变量可以改吗？可以加入我自己的目录吗？可以：

```shell
# 把这句话在命令行中执行就生效，或者追加到`~/.bashrc`配置文件中，然后使用`source ~/.bashrc` 生效
export PATH="$HOME/my_dir:$PATH"
```

>  注意: 不要随意更改这个变量，改错了，任何命令都执行不了。Linux 不会对你的危险操作做出提示，因为他的哲学就是，相信你知道自己在做什么。


## 创建我们自己的命令 {id="write-command"}

然后我们来创建一个自己的命令，你可以使用任何你熟悉的语言来编写。这里，我们使用 C 语言来编写一个经典的 Hello World 程序来演示。之所以使用 C 语言来编写是因为在 Linux 中大量的应用程序都是使用 C 来编写的，我们经常需要来编译安装这些软件，但往往不知所以然。所以使用 C 来编写，是想让大家理解原理。

创建一个名为`hello.c`的代码文件，代码如下:

```c
#include <stdio.h>

int main() {
	printf("Hello World\n");
	return 0;
}
```

然后我们来编写一个名为`Markfile`的文件，使用这个文件来告诉 gcc 编译器如何编译我们的代码，内容如下:

```Make
hello:hello.c
	gcc hello.c -o hello
install:
	mv hello /usr/bin/hello
```

解释一下文件内容：编译一个名为`hello`的可执行程序，其源代码文件是`hello.c`，使用的编译命令是`gcc hello.c -o hello`, 当我运行`make install`的时候，执行`mv hello /usr/bin/hello`命令。

所以，接下来，我们就可以来执行编译安装命令了：

```shell
make && make install
```

之后我们就可以在任意一个路径中，执行我们的`hello`命令了，然后会在终端输出`Hello World`并换行 ：

```shell
$ hello
Hello World
```

> 扩展开去说一句，为什么 C 语言那么强大，在 Linux 中我们会使用 Shell 来编写各种脚本而不是使用 C 语言呢？那是因为杀鸡不需宰牛刀。


相信通过这一小节，你应该更加深入的明白了 Linux 下命令的本质了。当你在工作学习中遇到`make && make install`这样的命令的时候，或者执行这样的命令出错的时候，就不会感到惊讶，而且容易排错了。

## 为什么 Windows 和 Linux 的软件不能通用 {id="why-window-and-linux-software-arent-interchangeable"}

其实不只是 Windows 和 Linux， Linux 和 Android，甚至是 MacOS 和 ipadOS 目前都不能通用(当然 Apple 有计划来实现这件事情，对广大的开发者真是福音)。

ELF 是英文 Executable and Linkable Format 的简称，换句话说它是 Linux 下可执行文件的格式。对的，可执行文件也是一种文件，既然是文件就会有它的编码格式。为什么 Linux 下的软件不能在 Windows 下运行？就是因为 Windows 的可执行文件使用的是 PE 格式，格式不同也就看着眼熟但不认识。

你可能会说，格式不同，那容易，格式转换一下就好，一个软件就不需要在 Linux 下写一套、Windows 下写一套了。我

们从 ELF 中找到答案，Executable and Linkable Format, 前面解释了 Executable，后面还有一个  Linkable 是什么意思。一个软件的编写，不是从 0 开始写的，会依赖大量的操作系统提供的框架、库。如果你用汇编写代码，那么不使用 Windows 或者 Linux 都可以，直接使用汇编器编译成二进制的 CPU 指令，交给 CPU 执行就好了。

但是 Linux 和 Windows 中的程序使用 C 或者 C# 之类的语言编写，大量依赖其他操作系统提供的能力。所以，类似于 Wine 这样的项目，期望在 Linux 上运行 Windows 程序，其工程难度之大让人望而生畏啊，到目前为止也并没有很大的突破。

## 总结 {id="summary"}

这篇文档介绍了Linux系统中命令的本质、位置、执行方式，以及如何创建自定义命令和为什么Windows和Linux的软件不能通用。

通过这篇文档，读者可以更深刻地理解Linux命令的工作原理，并且能够在遇到命令执行错误时更容易地进行排错。