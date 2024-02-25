# 进程基础

进程是操作系统中一个重要的概念，狭义的说，进程就是正在内存中运行的程序的实例。这篇文档详细描述了进程在 Python 中的应用。

## 什么是进程 {id="what-is-unix-processes"}

从 1970 年起，Unix 编程模型就已经以某种形式存在了。当时，Unix 在贝尔实验室闪亮问世，随之一起的还有 C 语言。进程是 Unix 系统的基石，因为所有的代码都是在进程中执行的。

比如说，我们执行一个简单的 Python 的脚本:
```Python
import time

print("Hello World")
time.sleep(20)
```
上面的脚本，在终端输出 `Hello World` 字符，并且休眠了 20 秒。然后我们可以再打开一个终端，观察这个进程:
```Shell
$ ps -ef | grep -v grep | grep hello
root     2646409 2622656  0 10:59 pts/2    00:00:00 python hello.py
```

这条 Shell 命令用于列出所有正在运行的 Python 进程，并显示它们的详细信息。输出显示，只有一个 Python 进程正在运行，该进程由 root 用户运行，PID 为 2646409，正在执行 hello.py 文件。具体每一列的输出如下表所示:

| 输出列               | 描述               |
|-------------------|------------------|
| `root`            | 表示该进程由 root 用户运行 |
| `2646409`         | 进程的 PID          |
| `2622656`         | 进程的父进程的 PID      |
| `0`               | 进程的优先级           |
| `10:59`           | 进程启动的时间          |    
| `pts/2`           | 进程的控制台设备         |
| `00:00:00`        | 进程使用的 CPU 时间     |
| `python hello.py` | 表示该进程正在执行的命令     |

## 进程的 PID {id="pid"}

通过上文，我们已经了解到每一个进程都有一个**唯一** PID（Process ID）。如下示例，展示了如何在 Python 脚本中输出当前进程的 PID:
```Python
import os

# 在 Linux 中会输出递增的 PID 序号
print("Current Process ID is:", os.getpid())
```
但是实际输出的 PID 可能不是连续的，如下图所示:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/oDJyQtxl9HrcpQNx47zP.png)

在Linux系统中，新进程的PID（进程标识符）通常由进程ID生成器按照一定的顺序分配，但这个顺序并不总是连续的。这是因为PID的分配受几个因素的影响，主要包括：

1. **PID 回绕**：
   当可用的PID空间用完后，Linux内核会回绕到其允许的最小PID值，并开始重新使用之前已经被分配和释放的PID。这意味着新的进程可能获得一个非连续的PID，即便有其他低数值的PID是可用的。

2. **并发**：
   如果系统上有多个进程同时创建新进程，分配的PID可能会因为竞态条件而不连续。

3. **保留的PID**：
   有些PID可能被系统保留供特定的服务或守护进程使用。

4. **PID 分配策略**：
   某些Linux发行版可能实现了特定的PID分配策略，如随机化分配PID，以增加安全性并减少可预测性。

5. **最大PID值**：
   系统上可能定义了最大的PID值。当达到这个值后，PID号会重新从一个较低的数开始分配。

6. **进程结束**：
   当进程结束时，它的PID会被释放，并且可以被随后创建的任何新进程使用。

另外，PID 的分配还受到 `/proc/sys/kernel/pid_max` 控制，这个文件中定义了 PID 的最大值。默认情况下，这个值可能是 4194304, 但可以设置得更高，以允许更多的并发进程。当内核必须分配新的PID时，它会选择一个大于最小值（通常是 300）的未使用的最小 PID。由于进程可以快速创建和销毁，所以PID的分配通常不会是连续的。

```Shell
# cat /proc/sys/kernel/pid_max
4194304
```

知道 PID 有什么用呢？在并发编程中，我们通常会将 PID 输出到日志文件中，以观察不同进程的输出日志。

通过 `ps -ef` 命令可以查看进程列表，我们发现有一个 PID 为 1 的进程:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/EL6QqiGolG1hgI6sSlC2.png)

PID 为 1 的进程，这通常是 init 进程（在现代系统上通常是 systemd），它是所有用户空间进程的父进程。

除此之外，还有一个特殊的进程，叫做 idle，其 PID 为 0。这是当没有任何可以运行的进程的时候，操作系统会执行的进程。这个进程不执行任何操作。但是通过用户态的 `ps` 命令是无法观察到这个进程的信息的，因为这个进程是内核态维护的。

## 进程的 PPID {id="ppid"}

每个进程除了有自己的 PID 之外，还有自己的父进程的 PID，称为 PPID。如下脚本所示, 如何获取当前进程的 ppid：
```Python
import os

print("Current Process ID is:", os.getppid())
```

所以，在现代操作系统中，进程是一个树状的结构，我们可以通过 `pstree` 命令来查看进程的父子关系:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/RxDjk4rM6n9lz2AFuoZC.png)

## 进程的文件描述符 {id="file-descriptors"}

在 Unix 哲学中指出，在 Unix 世界中，万物皆文件。这里的文件包括网络 Socket、管道、文件、设备都使用“文件”这一抽象概念。

那么什么是文件描述符呢？是一个文件的编号，而文件其实就是软硬件资源。比如下面的示例，打开了文件 `/etc/passwd`:
```Python
file = open('/etc/passwd', 'r')
print("File descriptor:", file.fileno())
file.close()
```
正常情况下，这个文件描述符会输出 3。进程打开的所有资源都会获得一个用于标识的唯一数字，这便是内核跟踪进程所用资源的方法。

我们再来看下面这个例子:
```Python
passwd = open('/etc/passwd', 'r')
print("File descriptor:", passwd.fileno())
hosts = open('/etc/hosts', 'r')
print("File descriptor:", hosts.fileno())

passwd.close()

dev_null = open('/dev/null', 'r')
print("File descriptor:", dev_null.fileno())
dev_null.close()
```
输出的结果是 `3 4 3`。这个例子有两个特殊之处:

* 所分配的文件描述符编号是尚未使用的最小的数值。
* 一旦资源被关闭，对应的文件的描述符编号会被重新利用。

<warning>
需要注意的是，已经关闭的资源是没有文件描述符的。
</warning>

每个 Unix 进程都有三个打开的资源，它们分别是标准输入（STDIN）、标准输出（STDOUT）、标准错误（STDERR）,它们被成为标准流。所以我们上面示例中的文件描述符是从 3 开始的。

如下程序所示，按序输出 0、1、2:

```Python
import sys

print("STDIN file descriptor:", sys.stdin.fileno())
print("STDOUT file descriptor:", sys.stdout.fileno())
print("STDERR file descriptor:", sys.stderr.fileno())
```
| 标准流    | 文件描述符 |
|--------|-------|
| STDIN  | 0     |
| STDOUT | 1     |
| STDERR | 2     |

## 进程的资源限制 {id="limit"}

上文我们已经说到，每个进程打开的文件都有一个递增的文件描述符。那么一个进程可以打开多少个文件呢？其实这是有限制的:

我们可以 Python 中输出这个限制的数额:
```Python
import resource

soft_limit, hard_limit = resource.getrlimit(resource.RLIMIT_NOFILE)

print("Soft limit on number of file descriptors :", soft_limit)
print("Hard limit on number of file descriptors :", hard_limit)
```
执行这个程序之后，输出如下:
```Python
$ python limit.py
Soft limit on number of file descriptors : 65535
Hard limit on number of file descriptors : 65535
```
`RLIMIT_NOFILE` 返回两个参数，分别是文件描述符的软限制和硬限制。硬限制是必须超级管理员才能修改的，如何修改呢?

使用超级管理员权限，修改 `/etc/security/limits.conf` 配置文件，修改如下:
```Text
* soft nofile 4096
* hard nofile 8192
```
在这里，`*` 表示这些限制应用于所有用户，`nofile` 是我们要修改的限制类型，`4096` 是软限制，`8192` 是硬限制。

软限制可以通过 `resource.setrlimit()` 来修改。

<warning>
需要注意的是，这里的示例代码只能在 Linux 中运行，在 Windows 需要改写。
</warning>

## 环境变量 {id="env"}

环境变量是包含进程数据的键值对，所有进程都从其父进程继承环境变量。创建一个文件，命名为 `env.py`,内容如下:
```Python
import os

message = os.environ.get('MESSAGE')
print(message)
```
运行下面的命令，读取设置的环境变量:
```Shell
MESSAGE='Hello World' python env.py
```
常见的应用场景如下：

* 如上面的例子所示，作为一种将输入传递到命令行程序中的通用方法，代价比解析命令行参数要小。
* 比如一些敏感数据，不适合硬编码在代码中的，比如密码，适合写入环境变量
* 一些环境紧密相关的数据，比如 IP、域名等信息

## 进程的参数 {id="arguments"}

所有进程都可以访问名为 ARGV 的特殊数组 `argv` 是 `argument vector` 的缩写，换句话说这是一个参数向量或者数组。它保存了在命令行中传递给当前进程的参数。下面是一个简单的例子，命名为 `args.py`，内容如下:
```Python
import sys

for i in range(len(sys.argv)):
    print(f"Argument {i}: {sys.argv[i]}")
```

执行这个脚本:
```Shell
$ python args.py --host=localhost --port=3987
Argument 0: args.py
Argument 1: --host=localhost
Argument 2: --port=3987
```
> 第一个参数永远是脚本的名字。

在 Python 的标准库中提供了 `argparse` 库来处理命令行参数:
```Python
import argparse

parser = argparse.ArgumentParser(description="An argparse example")
parser.add_argument('names', metavar='NAMES', type=str, nargs='*', help="A list of names")
args = parser.parse_args()

print(f"Hello, {', '.join(args.names)}!")
```

## 进程的名字 {id="named"}

在操作系统中，每个进程都有会自己的名字。如下的示例演示了如何在 Python 中获取进程的名字:
```Python
import os
import psutil

pid = os.getpid()
process = psutil.Process(pid)

print(process.name())
print(process.exe())
```

执行这个程序，输出如下:
```Shell
$ python name.py 
python
/usr/bin/python3.9
```

## 退出码 {id="return-code"}

所有进程在退出的时候都带有数字退出码（0-255），用于表明进程是否顺利退出。按惯例，退出码为 0 的进程被认为是顺利结束，否则则标识出现了错误。

下面的示例演示了如何指定退出码:
```Python
import sys

sys.exit(1)
```
> 在 Python 中，如何抛出的异常未被捕获处理，则返回非零值的退出码，一般是 1。

在 Linux 中，我们可以通过 `echo $?` 来输出上一个命名执行后留下的退出码。

## 子进程 {id="sub-process"}

在 Unix-like 的系统中，进程可以通过 `fork` 的系统调用创建子进程。这个子进程和原始进程是一模一样的。

**子进程从父进程处继承了其所占用内存中的所有内容，以及所有属于父进程的已经打开的文件描述符。但是子进程拥有自己唯一的 pid。**

创建一个名为 `create_subprocess.py` 的文件，内容如下:
```Python
import subprocess

result = subprocess.run(['ls'], stdout=subprocess.PIPE)
print(result.stdout.decode())
```
在这个例子中，我们使用 `subprocess.run()` 运行了一个 `ls` 命令，这个命令会列出当前目录下的文件和文件夹。我们通过 `stdout=subprocess.PIPE` 捕获了这个命令的输出，然后打印出来。

`subprocess.run()` 会等待进程完成，然后返回一个 `CompletedProcess` 对象。你可以从这个对象中获取进程的返回码 (`returncode`)、标准输出 (`stdout`) 和标准错误 (`stderr`)。

## 总结 {id="summary"}

这篇文档主要介绍了进程在操作系统，特别是Unix/Linux系统及Python编程中的概念与应用。主要内容概括如下：

1. **进程定义**：进程是正在内存中运行的程序实例，在Unix系统中尤为关键，所有的代码执行都在进程中进行。

2. **进程PID**：每个进程都有一个唯一的进程标识符（PID），可以通过`os.getpid()`在Python中获取当前进程的PID。PID可能由于多种原因（如PID回绕、并发创建进程、保留PID等）而不连续，并受到`/proc/sys/kernel/pid_max`设置的最大值限制。

3. **父进程PID（PPID）**：每个进程除了有自己的PID外，还有一个父进程的PID（PPID），可通过`os.getppid()`获取。进程以树状结构组织，可以用`pstree`命令查看父子关系。

4. **文件描述符**：在Unix系统中，打开的资源（包括文件、网络Socket等）通过文件描述符进行跟踪管理。标准输入、输出和错误流分别对应固定的描述符0、1和2。文件描述符是从最小未使用的数字开始递增分配的，且当资源关闭后可以被重新利用。

5. **资源限制**：进程能够打开的文件描述符数量受限于软限制和硬限制，可通过`resource.getrlimit()`查询，并使用`resource.setrlimit()`修改软限制。硬限制通常需要超级管理员权限才能更改，通过编辑`/etc/security/limits.conf`配置文件实现。

6. **环境变量**：进程继承自父进程的环境变量可通过`os.environ`访问，常用于传递信息给命令行程序或存储敏感数据。

7. **进程参数**：通过`sys.argv`数组可以获得在命令行中传递给进程的参数。Python标准库提供`argparse`模块处理复杂参数。

8. **进程名字**：进程拥有自己的名字，可使用`psutil.Process().name()`获取。

9. **退出码**：进程结束时返回一个退出码（0-255），0表示成功退出，非零值通常表示异常终止。在Python中使用`sys.exit()`指定退出码。

10. **子进程**：通过`subprocess.run()`等函数可以在Python中创建子进程，子进程从父进程中继承状态但具有独立的PID，并能捕获其执行结果。