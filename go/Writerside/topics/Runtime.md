# Runtime

Runtime（运行时），指的是程序运行所依赖的环境。这篇文档对比了 Go 的 Runtime 和其他语言的 Runtime 之间的区别，描述了 Go 的 Runtime 的相关特点。

### 不同语言实现Runtime的方式不同

不同语言其实现 Runtime 的方式不同、作用也不同。比如 Java 的 Runtime 就是 JVM（Java 虚拟机）。代码并不能直接编译成机器码运行，而是先编译成字节码，然后再 JVM 上解释或者二次编译运行。Runtime 还提供了比如 GC（垃圾回收）机制等功能。

对于 JavaScript 来说，Runtime 是 NodeJS 或者浏览器内核。由浏览器或者 NodeJS 来解释执行。

对于 PHP 来说，和  Java 是类似的，底层是 Zend 虚拟机，采用解释或者 JIT （即时编译）来执行代码。

### Go 语言中的 Runtime

Go 语言中的 Runtime 首先是 Go 源码中的一个包，它是和我们自己编写的程序一起打包进最终的可执行文件的。换句话说，Runtime 是 Go 程序的一部分, 它也会随着我们编写的程序一起运行。那么 Go 语言中的 Runtime 有什么用呢？

- 内存管理
- GC，垃圾回收
- 协程调度
- 屏蔽了不同操作系统之间的接口差异

除此之外，Go 语言的一些关键字的实现，实际上是通过再编译时转换成对应的 Runtime 提供的函数实现的，下表举例说明:

| 关键字  | 函数                             |
|------|--------------------------------|
| go   | newproc                        |
| new  | newobject                      |
| make | makeslice、makechain、makemap... |
| <-   | chansend1,chanrecv1            |

