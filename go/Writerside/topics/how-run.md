# Go 程序是如何运行的 

这一篇文档描述了 Go 程序的运行过程，从第一行汇编代码到最后执行  main 包的 main 方法，中间经过了那些步骤？通过这些步骤的拆解，我们可以更加深入理解 Go 程序的执行过程。

当我们编写 C、Java 或者更 Go 程序的时候，都会在一个名为 `main` 的方法中写下第一行代码。可是我们程序执行的第一行代码是从 `main` 开始的吗？显然答案是否定的。

Go 程序的第一行代码在 `runtime` 包下，我们来查找这样一类的文件:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/xiRvRrCe3G41AjMlRwPe.png" alt="img1" />

在 `runtime` 包下，我们看到这类以`rt0` 开头的文件，其中`rt` 是 `runtime` 单词的缩写，而 `0` 表示 runtime 的入口。比如说 `rt0_linux_amd64.s` 这个文件名中 `linux` 表示对应的操作系统，而 `amd64` 表示是 CPU 芯片的架构，而 `.s` 扩展名则表示文件中的代码是汇编语言编写的。也就是说，如果我们的程序运行在 Linux 上，并且 CPU 的架构是 amd64, 这会运行这个文件中的代码。接着我们来看一下这个文件中的代码，如下:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/F9HK9ojvV5oWVAqoCdEh.png" alt="img2"/>

所以，我们程序的第一行代码是在 `_rt0_amd64_linux`这个函数中，既 `JMP _rt0_amd64(SB)` 这行汇编。`JMP` 指令表示方法的调用，`_rt0_amd64` 则是方法名。如果是 Windows 系统呢？你可以可以找到类似的代码，如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/VcbF7Gn0GSz7F6oouw7x.png" alt="img3"/>

你发现了没？只要是 amd64 架构的 CPU，调用的都是同一个方法 `_rt0_amd64(SB)`。这个方法写在哪里呢？在 `runtime/asm_amd64.s` 文件中，方法内容如下：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/IGkLWpgqjArwm4yjcQLc.png" alt="img4"/>

这个方法就三行代码，第 16 行是将 参数 `argc` （这个是执行程序时候命令行中传递的参数的个数）放到寄存器中，第 17 行也是一样的道理，将参数 `argv`(这个是执行程序时候命令行中穿的参数）放到寄存器中。第 18 行则是函数的跳转，跳转到 `runtime.rt0_go(SB)` 方法中，这个方法就在当前文件中：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/mIVIE8lOTDGgb5V5EGwH.png" alt="img5"/>

这个是比较核心的方法，用来启动 Go 程序。第 150 行和第 151 行是将 `argc` 和 `argv` 两个参数放到栈上。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/24ptEumskzMyN0BQGuDG.png" alt="img6"/>

这段代码的作用是初始化名为 `g0` 的协程，这个协程是 Go 程序中第一个协程，是为了调度协程而产生的协程，所以它是不归协程调度器管理的（这时候还没有协程调度器呢）。

然后我们继续看下去，直到下面这一行关键的代码：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/14RR3c942QjRILzHcwRV.png" alt="img7"/>

程序执行到这里是第一次调用了 Go 语言编写的方法，这个方法编写在文件 `runtime/runtime1.go` 中:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/a50QLhb7IdnEW46vPWpO.png" alt="img7"/>

这个方法做了很多的前置判断，具体如下所示:

- 检查各种类型的长度
- 检查结构体字段的偏移量
- 检查 CAS 操作
- 检查指针操作
- 检查 atomic 原子操作
- 检查栈大小是否为 2 的幂次

跳出这个 `check` 方法，再往下看下去。下面的代码是将 `argc` 和 `argv` 两个参数从寄存器拷贝到 Go 语言的代码中去:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/7ffCyD2WZoV8ectMJbcD.png" alt="img8"/>

继续下去，下面三行代码, 就是三个方法的调用:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/n1aklXJSh4j6Ytq0slkd.png" alt="img9"/>

第 320 行：我们看到 `args` 这个方法，它的作用是对命令行中的参数进行处理，将 `argc` 赋值给 `argc int32`变量，将 `argv` 赋值给 `argv **byte` 变量。

第 321 行： 我们看到 `osinit` 这个方法，是为了判断操作系统的字长和 CPU 的核心数。在后面协程调度器初始化的时候需要用到。

第 322 行：我们看到 `schedinit` 这个方法，作用是初始化调度器。主要做了下面这些事情:

- 全局栈空间内存分配
- 堆内存空间的初始化
- 初始化当前系统的线程
- 算法初始化（map, hash)
- 加载命令行参数到 oa.Args
- 加载操作系统环境变量
- 垃圾回收器的参数初始化
- 设置 process 的数量

然后我们继续看下去:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/UxphIf3gf1Wje5dnRwQH.png" alt="img10"/>

第 325 行：取到 `mainPC` 这个主函数的地址，但是这个地址并不是 `main` 包中的 `main` 函数的地址。它其实是 `runtime` 包的 `main` 方法的地址。

第 327 行：`newproc` 这个方法是启动一个协程（就是`runtime.main` 方法）。但是需要注意，这里只是启动协程，这时候协程还没有运行，因为协程调度器还没有启动调度。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/0GvWdupinrYXvowdNsgA.png" alt="img11"/>

第 331 行，看注释是初始化一个 M，这个 M 是用来调度主协程的。到这里，调度器就起来了，然后 `runtime.main` 协程就可以开始执行了。

直到这里，我们的系统就已经有了两个协程了，分别是 `g0` 协程和 `runtime.main` 协程。`g0` 协程是所有其他协程的父协程，是用来启动调度器。而 `runtime.main` 则放入了调度器等待被调度执行。这个 `runtime.main`方法编写在文件 `runtime/proc.go` 文件中，内容如下:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/HeKFHIYaEu5Eh23mxDhy.png" alt="img12"/>

首先在第 199 行，执行了 `doInit` 方法，这个方法用来初始化操作。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/vo42pcmMdhVu8gLvaths.png" alt="img13"/>

接着在第 209 行，执行了 `gcenable` 方法。也就是说，就此启动了 GC 垃圾回收。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/FCUKQrxPiZKfPOVhzAMB.png" alt="img14"/>

在接下来，第 249 行，定义了一个名为 `main_main` 的方法，这个方法就是我们编写的 `main` 包下面的 `main` 方法。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/YkLgmlhWlSo4bQWMsZ31.png" alt="img15"/>

我们来看到这个 `main_main` 的定义处，看到第 132 行的注释 `//go:linkname` 。这个注释是有意义的，是告诉链接器链接我们编写的 `main` 包中的 `main` 方法。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/yCf7sXEWlJVQQhwqPMhf.png" alt="img16"/>

到此，我们编写的 `main` 方法就开始执行了。