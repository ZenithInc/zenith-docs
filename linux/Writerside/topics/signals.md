# Linux 信号

什么是 Linux 信号？是一种进程通信机制。就像十字路口的信号灯一样，当不同的车辆(类似进程)收到信号之后，会按照信号预定的含义行事，红灯停绿灯行。
然而，也有车辆违章行事或者事出紧急。进程在收到信号之后，也可以改变默认的行为，做自己想做的事情。

## 什么是信号 {id="what"}

在 [《Linux processes and signals》](https://www.bogotobogo.com/Linux/linux_process_and_signals.php)一文中有如下描述:

> **Signal**is a notification, a message sent by either operating system or some application to our program. Signals 
> are a mechanism for one-way asynchronous notifications. A signal may be sent from the kernel to a process, from a 
> process to another process, or from a process to itself. Signal typically alert a process to some event, such as a 
> segmentation fault, or the user pressing Ctrl-C.



信号一种通知，由操作系统或一些应用程序将一条消息发送给我们的程序。信号是一种单项的异步通知机制。一个型号可能从内核发送给进程，可能从一个进程发
送给另一个进程，或者从一个进程发送给他自己。信号通常用来通知进程发生了一些事情，比如说段错误，或者用户按下了 `Ctrl-C` 组合键。

## 信号的分类 {id="category"}

Linux 信号可以分为下面两种类型:

- `Maskable`: 信号默认的行为可以被用户更改或者忽略。
- `Non-Maskable`: 信号不能被用户更改或忽略，比如说硬件发生了不能回复的错误。

## 常用的信号 {id="common"}


Linux 内核定义了大概 30 个信号。每一个信号都有一个唯一标识的数字，从 1 到 31。信号没有任何的参数，他们的名字都解释了他们的作用。比如说， `SIGKILL`
这个信号的编码是 9，他告诉指定的进程有人在尝试结束他。

你可以通过 `kill -l` 命令来查看自己的 Linux 发行版支持的所有的信号:
```bash
$ kill -l   ## 示例使用的 Linux 发行版是 CentOS Linux release 7.6.1810 (Core)
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP    
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1    
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM    
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP    
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ    
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR     
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3 
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8 
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7 
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2 
63) SIGRTMAX-1  64) SIGRTMAX
```
我们并没有必要记住所有的信号量，但是对某些常用的信号量应该知悉，列举如下表:

| 信号量        | 含义                                           |
|------------|----------------------------------------------|
| SIGCHLD-17 | 由子进程发送给父进程，告知子进程已经结束                         |
| SIGQUIT-3  | 表示终端推出                                       |
| SIGTERM-15 | 终止进程，由进程决定什么时候关闭，优雅的关闭                       |
| SIGKILL-9  | 终止进程的运行，不会等待                                 |
| SIGHUP     | 迫使进程重新读取配置文件，挂起检测                            |
| SIGUSR1    | 用户自定义的信号量1                                   |
| SIGUSR2    | 用户自定义的信号量2                                   |
| SIGWINCH   | 表示窗口(Window)发生改变(Change)                     |
| SIGINT     | 和 SIGKILL 信号的区别在于其不能被捕获和忽略，使用 `Ctrl+C` 发送该信号 |

## Shell 程序信号处理 {id="handler"}

在 Shell 中，使用 `kill` 命令来向进程发送信号。如果不指定信号量 `kill <pid>` , 默认发送的是 `SIGTERM` 信号，允许进程自身去决定结束的时机
，一般进程会利用接受到信号到结束进程之间的时间来处理数据，完成请求等操作，是的进程可以优雅的结束。

下面这个程序，接受并处理了 `SIGINT` 信号:
```bash
#!/bin/bash
## 定义信号处理函数，当接受到信号的时候执行该函数
function handler()
{
  echo Received signal: SIGINT
}
## 注册 SIGINT 信号量的处理函数
trap 'handler' SIGINT
## 死循环，等待信号的接收
while true;
do
  sleep 1
done
```
执行上面这个程序，按下 `Ctrl+C` 会向这个 Shell 程序的进程发送一个 `SIGINT`  的信号，屏蔽了默认的该信号的处理程序，即不会退出该程序。如果需
要推出该程序，按下 `Ctrl+Z` 中断程序的执行，然后使用 `kill -9` 向该进程发送终止信号。

## 参考资料 {id="reference"}

1. [Linux Programmer's Manual —— SIGNAL(7)](https://www.man7.org/linux/man-pages/man7/signal.7.html)
2. [What are Linux signals?](https://www.educative.io/edpresso/what-are-linux-signals)
3. [Linux processes and signals](https://www.bogotobogo.com/Linux/linux_process_and_signals.php)
4. [《Linux Shell 脚本攻略》第三版](https://book.douban.com/subject/30172987/) —— 第十章 管理重任
5. 慕课网《新版Nginx1.17体系深度精讲 给开发和运维的刚需课程》Linux的信号量管理
