# Nginx 架构

在对 Nginx 有了大致的了解之后，我们再来从架构层面来看看它。

所有的 Unix  应用都是基于多进程或者多线程的。他们之间的主要区别在于进程之间内存是隔离的，而线程之间内存是共享的。所以多进程的架构上下文的切换消耗更多的资源，而多线程相对更少。

## Nginx 的架构 {id="architecture"}

Nginx 是采用多进程单进程的架构。而采用多进程或多线程的架构的应用程序，其原因大致有以下两点:

- 可以在同一时间内使用更多的 CPU 核心。
- 线程或进程可以非常轻松的在同一时间内做更多的操作(对于 Nginx 来说，就是可以建立更多的请求连接，为此提供服务)。

那么 Nginx 为什么不采用多线程的架构呢？这是因为进程之间是内存隔离的，而线程之间是采用共享内存的。如果采用多线程，会因为程序上可能的不健壮导致内存地址空间越界等问题。而 Nginx 往往是运行在服务的临界点(靠近用户的位置),对稳定性由更高的要求。

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/rWqB0Zbv7PYanDbkZDEX.png)

需要说明的是，Master 进程并不负责转发请求给 Worker 进程，这部分工作是由系统内核采用 epoll/select 模型来完成的。采用这样的设计，对编码提出了更高的要求。更为具体的内容，可以参考如下说明:

> nginx uses multiplexing and event notifications heavily, and dedicates specific tasks to separate processes. Connections are processed in a highly efficient run-loop in a limited number of single-threaded processes called `worker`s. Within each `worker` nginx can handle many thousands of concurrent connections and requests per second.    —— 《[Inside NGINX: How We Designed for Performance & Scale](https://dzone.com/articles/inside-nginx-how-we-designed)》



Nginx 使用了多路复用技术和大量的事件通知，并且将特定的任务分配给不同的进程。连接在被称之为 workers 的有限数量的单线程的进程中高效的循环运行。这使得每个 Nginx 工作进程可以在每秒内处理成千上万的并发连接。

> Processes and threads consume resources. They each use memory and other OS resources, and they need to be swapped on and off the cores (an operation called a context switch). Most modern servers can handle hundreds of small, active threads or processes simultaneously, but performance degrades seriously once memory is exhausted or when high I/O load causes a large volume of context switches. —— 《[Chapter “nginx” in “The Architecture of Open Source Applications”](https://www.aosabook.org/en/nginx.html)》


进程和线程都需要消耗资源。他们在上下文切换的时候，都需要内存以及其他的系统资源。很多的现代服务器都能处理启用线程和进程去处理成败的请求连接，但当发生内存不足或者切换上下文加载较大的磁盘卷、高 I/O 负载的时候，其性能是比较差的。

> The common way to design network applications is to assign a thread or process to each connection. This architecture is simple and easy to implement, but it does not scale when the application needs to handle thousands of simultaneous connections.  —— 《[Inside NGINX: How We Designed for Performance & Scale](https://dzone.com/articles/inside-nginx-how-we-designed)》


一种通用的网络程序设计是注册线程或进程去轮询这些连接。这种架构是简单而且易于实现的，但在处理成千上万的请求的时候，它是不易于伸缩的。

## Nginx 是如何工作的 {id="how-nginx-works"}

我们知道 Nginx 的进程分为 Master 进程、Worker 进程以及一些辅助进程(比如缓存控制)。其中 Master 进程做了如下这些事情:

- 加载和验证配置的有效性
- 创建、绑定和关闭 Sockets 套接字
- 创建、终止以及维持基于配置的数量的工作进程
- 在不中断服务的情况下重新加载配置
- 控制二进制文件热更新(载入新的二进制如果失败则回滚)
- 重新打开日志文件
- 编译嵌入式的 Perl 脚本

一个典型的 HTTP 请求在 Nginx 中的生命周期如下:

1. 客户端发起 HTTP 请求
2. Nginx 的核心会根据配置中与请求匹配的位置选择合适的处理程序
3. 如果存在相应的配置，则负载均衡到代理的上有服务器
4. 相关的处理程序完成了它的工作之后，将每个输出缓冲区传递给第一个过滤器
5. 第一个过滤器处理完成后将输出传递给第二个过滤器
6. 第二个过滤器将输出再传递给第三个过滤器(以此类推)
7. 最终请求的响应将传递给客户端

Nginx 提供了高度可定制化的模块机制，第三方的开发者可以基于这个机制开发出符合自身业务的模块通过编译插入 Nginx(目前应该是不支持动态加载第三方模块)。然后,Nginx 在特定的时机会去类似于管道机制的去执行这些插入的模块。比如下面这些事件节点:

- 读取和处理配置文件之前
- 主配置文件加载完成之后
- 当服务初始化(完成了对 host 和端口的监听)之后
- 当 server 的配置合并到主配置的时候
- 当 master 进程开启或结束的时候
- 当 workers 进程开启或结束的时候
- 当处理请求的时候
- 当上游服务器完成处理返回的时候
- ......

## Nginx 架构中的经验教训 {id="lessons-learned-from-nginx-architecture"}

这部分内容来源于《[“nginx” in “The Architecture of Open Source Applications”](https://www.aosabook.org/en/nginx.html)》，我觉得有必要列出来，让大家看到。

> When Igor Sysoev started to write nginx, most of the software enabling the Internet already existed, and the architecture of such software typically followed definitions of legacy server and network hardware, operating systems, and old Internet architecture in general. However, this didn't prevent Igor from thinking he might be able to improve things in the web servers area. So, while the first lesson might seem obvious, it is this: there is always room for improvement.


当 Igor Sysoev 在开始写 nginx 之后，大部分网络软件已经存在，这一类的软件的架构都遵循传统的网络硬件、服务以及操作系统旧的网络架构。然后，这并没有阻止 Igor 去思考更多的改进 web 服务性能。因此，虽然第一个教训显而易见，但它总是这样的： **总是有改进的空间** 。

> With the idea of better web software in mind, Igor spent a lot of time developing the initial code structure and studying different ways of optimizing the code for a variety of operating systems. Ten years later he is developing a prototype of nginx version 2.0, taking into account the years of active development on version 1. It is clear that the initial prototype of a new architecture, and the initial code structure, are vitally important for the future of a software product.


考虑到更好的网络软件的想法，Igor 花了很多时间开发初始代码结构，并研究不同的方法来优化各种操作系统的代码。 10年后，他正在开发 nginx 版本2.0的原型，考虑到多年来在版本1上的积极开发。 很明显，新架构的初始原型和初始代码结构对于软件产品的未来至关重要。

> Another point worth mentioning is that development should be focused. The Windows version of nginx is probably a good example of how it is worth avoiding the dilution of development efforts on something that is neither the developer's core competence or the target application. It is equally applicable to the rewrite engine that appeared during several attempts to enhance nginx with more features for backward compatibility with the existing legacy setups.



另一点值得一提的是，发展应当集中。 版本的 nginx 可能是一个很好的例子，说明如何避免在既不是开发人员的核心能力，也不是目标应用程序的东西上削弱开发努力是值得的。 它同样适用于重写引擎，出现在几次尝试，以增强 nginx 与现有的遗留设置更多功能的向下兼容。

## 参考文档 {id="reference"}

1. 《[Inside NGINX: How We Designed for Performance & Scale](https://dzone.com/articles/inside-nginx-how-we-designed)》
2. 《[“nginx” in “The Architecture of Open Source Applications”](https://www.aosabook.org/en/nginx.html)》

