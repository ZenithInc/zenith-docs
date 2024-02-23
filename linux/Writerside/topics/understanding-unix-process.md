# 读《理解Unix进程》一书 

这一篇文档是我阅读这本书的笔记，这是一本非常薄的书，言简意赅，对于 Unix 进程这一主题进行深入的分析，使用 Ruby 语言。

这一篇来讲解面试中经常会问起的进程，它们是什么？以及它们之间是如何通信的。

### 什么是进程 {id="what-is-process"}

什么是进程？很多人和我一样，好像似懂非懂。因为从事高级编程，所以对于这些概念而言，也仅仅是概念。比如对于 PHPer 来说，每一个 PHP 脚本就是一个进程。PHP 还有一个名为 PHP-FPM 的进程管理器，用来为每一个请求分配一个进程去处理。

这里我推荐一本书，名为 [《理解 Unix 进程》](https://book.douban.com/subject/24298701/)。这是一本很小的册子，花三两个小时就可以看完它，看完它你会对进程有更多的理解。如果你的时间并不是那么充裕，那么也可以看本文接下来的内容，会对这本书中的核心内容做一些摘录，相信也可以让你了解进程。

### 进程和程序之间的关系 {id="difference-between-process-and-program"}

程序是什么？程序是代码，而代码是存储在磁盘中的文本文件中的内容。这些文件并不会自动运行，因为它们和其他的文本文件没有什么区别。

CPU 要执行程序的时候，必须要把代码加载到内存中，然后进一步将内存中的数据加载到高速缓存中，最后从缓存中读取指令并运行。

这就是计算机系统中的多级存储机制，为什么不直接从硬盘中读取速度呢？因为 CPU 的运行速度远远大于硬盘，等不起啊。

另外，**所有的程序都是指令和数据的集合。**除了程序本身外，还需要有数据。因为程序只会做三件事情: **输入、运算、输出。**所以，需要输入数据、对数据进行运算、在将运算后的结果数据输出到文件或者终端中。而在运行期间的数据，也就是我们程序中的变量，有同样是存储与我们的内存之中的。

**所以，我们把内存中的程序叫做进程，把进程也称之为程序的实例。为什么说是实例呢？因为一个程序可以开启多次，每一次都在不同的进程中，虽然使用同样的代码，但它们的数据并不通用。**
**
所以，在 《理解 Unix 进程》一书中将进程称之为 Unix 之本，因为所有的代码都是运行在内存中的进程中的。
```ruby
$ ruby -e "p Time.now"
```
> 《理解 Unix 进程》这本书中的所有示例都是使用 Ruby 语言编写的，运行实例首先需要安装 Ruby 的解释器。在 Ubuntu 下可以使用 `sudo apt install ruby -y` , 而在 CentOS 下，可以使用 `sudo yum install ruby -y` 安装。


上面的代码在实行的时候，就是一个内存中的进程。执行结束之后，这个进程就被操作系统销毁了，在内存中也就不复存在了。在操作系统中运行的每一条命令、每一个程序都是一个进程，比如使用 `cd` 切换目录，或者运行 MySQL 这样的大型软件系统都是如此。

### 进程皆有标识 {id="process-flag"}

在系统中运行的进程都有一个唯一的进程标识符，我们称之为 pid（process id）。就像我们数据库系统中的用户 ID 一样，本身并没有一个意义，只是一个自增长的序列编号而已。pid 也是在内核中用来标记进程的一个自增长的编号。
```shell
$ ruby -e "puts Process.pid"
3155   # 输出一定是和我不一样的，但应该是递增的整数
```
上面的代码输出了程序自身在运行时候的 pid, 这是操作系统在创建该进程的时候分配的一个整形的数值。从1 开始递增。所以，有一个 pid 为 1 的进程最为特殊，我们通常称之为初始化进程。在 CentOS7/8 中，这个进程为 `/usr/lib/systemd/systemd` 。

Ruby 语言中的 Process.pid 实际上是对操作系统中的 `getpid` 这个系统调用的封装，我们可以通过 `man 3 gitpid` 来查看手册。另外，下文给出 C 语言获取 pid 的代码示例:
```c
#include <stdio.h>
#include <unistd.h>

int main() {
        int pid = getpid();
        printf("Pid is %d.\n", pid);
        return 0;
}
```
在 Bash Shell 中，我们也可以通过 `$$` 符号来获取 pid，示例如下:
```shell
#!/bin/bash

echo $$
```
在实际的环境中，有一些长期驻守在后台的进程，会将这个 pid 进程号写入一个文件，以便其他程序调用。我们也称呼这样的文件为 pid 文件。比如说 Nginx 、MySQL 都会如此。

### 进程皆有父 {id="parent-process"}

在 Unix/Linux 系统中，我们可以通过系统调用 `fork` 来启动新的进程，A 进程启动 B 进程，那么 B 就是 A 进程的子进程。除了特殊的 pid 为 1 的进程外，都会有父进程。我们可以通过 `pstree` 命令来查看进程的父子关系, 如下图所示:
![image.png](https://cdn.nlark.com/yuque/0/2020/png/502915/1599536949611-b1daf658-c0eb-473c-a512-3649e2133d9e.png#align=left&display=inline&height=67&originHeight=201&originWidth=649&size=20783&status=done&style=stroke&width=216.33333333333334)
进程的 id 称之为 pid, 进程的父亲 id 称之为 ppid。我们使用 Ruby 来演示如何输出 ppid:
```shell
ruby -e "puts Process.ppid"
```
我们可以通过 `ps` 命令来查看父进程的相关信息:
```shell
$ ps -p $(ruby -e "puts Process.ppid")
    PID TTY          TIME CMD
   2798 pts/0    00:00:00 bash
```
可以通过 `getppid(2)` 来查获取父进程id的系统调用。

### 进程皆有文件描述符 {id="process-file"}

在 Unix 中，每当进程打开一个文件，就会为这个文件赋予一个文件描述符，它是一个整形递增的数值。这样做的目的，是为了系统内核可以跟踪这些被打开的文件。

但是有三个特殊的文件，被称为标准流，分别是标准输入(STDIN)、标准输出(STDOUT)以及标准错误(STDERR)。他们分别对应着 0、1、2 三个固定的文件描述符。

每一个被创建的进程都会自动拥有（打开）这三个资源。为什么要这么做呢？拿 STDIN 举例，为了能够支持键盘，你需要指定一个键盘的驱动程序。如果你要在屏幕中输出 Hello World ，你需要知道并控制屏幕的像素。但是有了标准流之后，你就不需要这么做了。**所以说，标准流的机制屏蔽了硬件设备的复杂性(是对众多硬件设备的封装或抽象)。**

下面，我们使用程序打开一个文件，看看文件描述符长什么样子？其实和 pid 以及 ppid 一样:
```ruby
passwd = File.open('/etc/passwd')
puts passwd.fileno
passwd.close
```
我们创建一个名为 `demo.rb` 的文件，输入以上内容，然后使用 `ruby demo.rb` 运行它，会在终端输出一个整数值，这个数值就是所谓的文件描述符。对于运行中的进程而言，它就是文件的 ID。对于操作系统而言，会对每一个被进程打开的文件编号，编号的规则是大于 3 并且没有被使用的最小数值。

如果你在 `demo.rb` 中追加一行代码并运行，就会报错，代码如下:
```ruby
passwd.close
puts passwd.fileno
```
错误输入如下:
```
Traceback (most recent call last):
        1: from test.rb:4:in `<main>'
test.rb:4:in `fileno': closed stream (IOError)
```
当我们的文件流被关闭( `passwd.close` )， 操作系统就会回收这个文件描述符，以供其他需要的进程使用。

### 进程皆有资源限制 {id="process-limit"}

之所以出现进程的概念，很大程度是也是为了对众多运行的程序进行统一的管理，避免单一的程序占用过多的资源，以至于其他的程序无法正常运作。所以说，进程也皆有资源限制。

举例说，我们上文提到系统会为每个被进程打开的文件分配一个动态的文件描述符，但是这并不是无限制的。默认情况下，是 1024。我们可以通过 Linux 下 `ulimit` 命令来查看这个限制的数值:
```shell
$ ulimit -n
1024
```
然后我们再来看下面这个程序片段:
```ruby
$ ruby -e "puts Process.getrlimit(:NOFILE)"
1024
262144
```
这句代码输出了两个值，第一个 1024 只是的程序最多能够打开的文件数量的软限制，而第二个 262144 指的是程序最多能够打开的文件数量的硬限制。那么软限制和硬限制有什么区别呢？

软限制程序自身也能够更改，而硬限制呢？除非是超级管理员或者具有超级管理员的权限才能够修改。
```ruby
Process.setrlimit(:NOFILE, 4096)
puts Process.getrlimit(:NOFILE)  // 输出 4096 4096
```
此外，还有其他的很多的限制，比如说限制文件创建的大小。

### 进程皆有环境和参数 {id="process-env"}

这里的环境指的是环境变量，即子进程会继承父进程中的环境，即环境中的变量。我们举例说明:
```shell
export MESSAGE='Hello World' && ruby -e "puts ENV['MESSAGE']"  ## 输出 Hello World
```
相对于解析命令行参数，解析环境变量的代价会小一些。那么如果要获取命令行的参数如何做呢？
```ruby
puts ARGV
```
我们可以将上面的文本内容保存在一个名为 `test``.py` 的文件中，然后执行它
```shell
$ ruby test.rb foo bar -va
foo
bar
-va
```
这会将所有的命令中的参数全部输出到终端。

### 进程皆有名 {id="process-name"}

每一个进程都有名字，默认是程序的文件名。我们可以在运行中修改这个名字，铜鼓 `$PROGRAM_NAME` 这个变量:
```ruby
puts $PROGRAM_NAME
$PROGRAM_NAME = "Process Demo"
puts $PROGRAM_NAME
```
通过在运行时修改进程的名字，还可以达到进程间通信的目的。

### 进程皆有退出码 {id="exit-code"}

当进程结束的时候，可以使用一个退出码来告诉接下来要运行的程序自己的运行状态。这个退出码需要在 0-255 之间。我们以 Shell 中的内建命令 `cd` 为例:
```shell
$ cd not_exists_dir
-bash: cd: not_exists_dir: No such file or directory
$ echo $?
```
如果是进入一个不存在的文件夹，该程序就会返回 一个 1 的退出码。如果目录存在，则返回为 0。

在 Ruby 中，我们可以通过 exit 来指定退出码:
```ruby
exit 2
```


## 参考资料 {id="reference"}

1. [《理解 Unix 进程》](https://book.douban.com/subject/24298701/)
2. [《Linux 就是这个范儿》](https://book.douban.com/subject/25918029/)