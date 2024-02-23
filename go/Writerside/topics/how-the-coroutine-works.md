# 协程是如何运行的

这篇文章通过分析 Go 的源码，来深入理解协程的本质。通过对协程运行原理的了解，我们可以更加深刻理解继承，运用协程。需要注意的是，本文会涉及 Go 的源码，使用的 Go 版本为 v1.18。

## 一个简单的协程的示例 {id="example"}

我们需要编写一个简单的协程的实例，然后通过这个跟踪这个示例的逐步执行，来理解协程的运行流程。
```go
package main

import (
	"fmt"
	"time"
)

func wrap() {
	inner()
}
func inner() {
	fmt.Println("You are my sunshine!")
}
func main() {
	go wrap()
	time.Sleep(time.Second)
}
```
程序很简单，就是在 `main` 函数中调用了 `wrap` 函数，在 `wrap` 函数中调用了 `inner` 函数。通过 `go` 关键字将 `wrap` 函数包装成为一个协程。最后 `time.Sleep` 是为了让协程能够在 `main` 执行结束之前完成。

## 协程本质上是一个结构体 {id="coroutine-is-struct"}

协程的结构我们通过一张示例图来总览一下，之后再仔细分析:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/rhzQdPQFX8IPH65DV6En.jpeg)

协程本质上就是一个结构体，在 `runtime/runtime2.go` 这个文件中定义，截图如下:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/ntL4oWQZ8pA3Xawz8PTD.png)

这个名为 `stack` 的成员也是一个结构体，源码的截图如下:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/u1T9dkxpNXLkoSNKd3xp.png)

在这个名为 `g` 的结构体中，第一个属性是 `stack` 是一个栈的结构。我们观察到，它有两个成员，是一个名为 `lo` 的原始指针，指的是栈的低位指针，另外是一个名为 `hi` 的原始指针，指的是栈的高位指针，他们存储的是栈帧信息。

我们可以上开始的示例代码的第 9 行打一个断点，来观察这个栈的信息:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/QqbKqGK8BHOw66oJ1fWt.png)

通过上面的截图，这个程序一共有三个栈帧，其中还有一个名为 `runtime.goexit` 的栈帧。

除了 `stack` 的成员外，还有一个名为 `sched` 的成员，其类型是 `gobuf` 的结构体，代码截图如下：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/zHcmJpQ9qWU6nbGPV6fW.png)

我们来看几个重要的成员，其中：

- `sp` 是一个原始指针，它是一个栈指针(**S**task **P**ointer), 指向了当前协程栈指向哪个栈帧，比如上面 Debugger 的截图，它指向的是 `main.wrap` 这个栈帧。
- `pc` 是程序计数器(**P**rogram **C**ounter)，指的是程序运行到了哪一行代码。

除了 `stack` 以及 `sched` 之外，还有一个名为 `atomicstatus`指的是协程的状态。另一个名为 `goid`的成员指的是协程的编号。

## 协程是运行在线程之上的 {id="coroutine-on-the-threads"}

协程并不是独立存在的，而是运行在协程之上的。在 Go 中，线程也是一个抽象的结构体，被定义在 `runtime/runtime2.go` 中，名为 `m`：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/mZiaKdYXdjlDcp8w4PGs.png)

这个结构体有非常多的成员，我们先挑一些重点的来解释：

- `g0` 就是 Go 程序启动的第一个协程，原始的协程，用来启动其他协程
- `crug`指的是当前正在执行的协程
- `id` 线程的唯一编号
- `mOS` 这个结构体记录了每一种操作系统对协程额外的描述信息（每种操作系统对线程的实现是不一样的）

下面的截图就是 `mOS` 不同的操作系统下额外信息的定义:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/94QnFbMzyrv7TnLatUQY.png)

## 单线程循环 {id="single-thread-loop"}

在 Go 还没有正式发布以前，也就是 v0.x 版本中，协程 都是在单个线程上去执行的。拿我们上文的示例程序而言，协程的执行过程如下图所示:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/Exbt5Mrx0Z8kTVnhDtqY.jpeg)

这张图描绘的是协程在单个线程中的执行过程，是一个循环，执行的逻辑从 `schedule` 一直到 `goexit`。这些方法的调用信息被记录在 g0 stack 中，之所以不是 g stack 有两个原因：

- 当我们的协程还没有在 runnable queue 中拿到一个协程去执行的时候，还没有 g stack。所以线程循环在一开始使用的是 g0 stack。
- 普通协程的栈，只能记录业务方法的调用信息。

既然线程是从一个名为 `schedule()` 的方法开始执行，那我们来看一下这个方法，它被定义在 `runtime/proc.go` 文件中，其中关键的在下面这行代码:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/VGDvAyhvI2eTwo6XEW3e.png)

`gp` 这个变量指的是即将要执行的协程，接下去大部分的代码都是为了去拿到这个要执行的协程，如下:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/shb53qLLEtSro6ozoC7Q.png)

然后在这个方法的最后，执行了 `execute()` 方法:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/pxYVn5VZ9rAMGCowjLWR.png)

我们再来看这个 `execute` 方法中做了些什么:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/08Rit3yV3UyJfBojsu6j.png)

传入的是 `gp` 变量，也即是即将要执行的协程。然后是给 `gp` 里面的一些字段赋值。最后调用了 `gogo()` 方法，如下截图:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/FURb5Um5oLsr6tztXYs7.png)

但是如果你去看这个方法的话，发现其只有一个函数的声明，这说明这个方法的实现是汇编语言编写的:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/ugP58WJUXi7krnn5oSBA.png)

既然是汇编实现的，不同的平台肯定是不一样的。所以在 Go 源码中，为每个平台都编写了 `gogo` 方法的实现。我们来看 amd64 平台的实现, 文件位于 `runtime/asm_amd64.s`：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/DTGBU3q71aDkbX2gibwF.png)

根据注释 `// func gogo(buf *gobuf)` 传入的参数是一个 `gobuf` 的结构体，根据上文中协程的结构图我们知道这个结构体中是 `sp` 指针和 `pc`指针。看这个方法的第 380 行，使用 JMP 汇编指令跳转到了下面的`gogo`方法，方法截图如下:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/6cSLAlIvgNuU0uxCqmcm.png)

这里面关键的在第 386 行，往协程栈中插入了一个栈帧，就是在上面的图示中的 `goexit` 方法。之所以插入这个方法， 是为了协程执行完成之后可以执行这个方法。

然后是第 394 行和第 395 行，这两行是跳转这个线程执行执行的程序计数器(`gobuf_pc` 就是 `gobuf` 结构体中的 `pc`程序计数器），这个记录的是我们协程运行到了哪一行代码, 存入 `BX` 寄存器，然后使用 `JMP` 指令跳转到 `BX` 寄存器中去，去执行我们从 runnable 队列中拿到的协程，这时候用的是协程自己的 g 协程栈，这是因为每一个队列中取出的协程都需要记录自己的执行现场的信息。

最后执行`goexit` 方法，在 `runtime/stubs.go` 文件中。从文件名也可以看出来，这只是桩代码，而实际代码是使用汇编写的，我们再来查看 amd64 的汇编实现，在文件 `runtime/asm_amd64.s` 中：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/4WQyaOinWxqH5Bc1DPiR.png)

这里面最重要的是第 1570 行代码，调用了 `runtime.goexit1(SB)` 这个方法。这个方法是使用 Go 来实现的，位于 `runtime/proc.go` 文件中：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/C1DBLK4IsGnH37bVaEGS.png)

从第 3442 行到第 3448 行只是调试用的，真正执行的是 `mcall(goexit0)`，这个 `mcall` 会将 g 协程栈切换到 g0 协程栈中，并执行 `goexit0` 方法。这个方法如下:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/TkxnO7acS98VLnsvLgyd.png)

我们看第 3464 行到第 3474 行代码，因为协程已经执行完成，所以需要更改协程的各种参数以及状态。然后再这个方法的最后，又调用了 `schedule()` 方法, 实现循环从队列中取出协程并执行：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/PNyRk7YSHx1KEg1OUI4O.png)

上面的过程我们也可以用下面这样更为简化的图来表示:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/oLI2CZJWZTfMSRbM797A.jpeg)

## 多线程循环 {id="multiple-threads-loop"}

我们现在的 CPU 都是多核多线程 CPU，单线程无法充分利用 CPU 性能。所以，从 Go1.0 开始采用了多线程循环的执行方式。下面的图示以两个线程为例，演示了多线程循环的结构以及执行过程:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/YKIrgbzuuKaURnqsmbEK.jpeg)

多个线程从队列中获取协程同时执行，就会存在并发问题，一个协程可能会在多个线程中执行。所以，这个全局的线队列就需要加上锁。我们使用简化的图如下所示：

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/5LIZaKpYFfRmcy4hC1hB.jpeg)

## GMP调度模型 {id="gmp-schedule-model"}

在上文中，我们引入了多线程循环。但是多线程循环存在着两个问题：

- 协程是顺序执行的，不能并发
- 多线程并发时，会抢夺协程多列的全局锁

为了解决这两个问题，Go 引入了 GMP 调度模型，在线程循环的基础上新增了 P(Processer, 处理器)。因为只有一个全局队列，所以多线程处理势必要引起频繁的锁的竞争冲突。GMP 模型如下图所示：

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/BEywkHWC7tbpldEgKZ30.jpeg)

所以为了解决这个问题，引入了本地队列的概念。原来每次从队列中获取单个协程，现在改为批量获取协程。原本全局共享一个队列，现在除了全局队列外，引入本地队列。每个线程有限读取本地队列，如下图所示:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/z3j3S0Hl6lILfR6a2sB8.jpeg)

那么负责批量从全局队列中取协程存入本地队列的就是 GMP 中的处理器，它也是一个结构体，我们可以在`runtime/rutnime2.go` 中看到它的定义:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/buOtKHSD9Ak7BjiI8pNA.png)

其中比较重要的几个成员，一个是 `m` ，它是一个原始指针类型，指向的是其要处理的线程结构。

第 615 到第 618 行，我们可以看出，从注释我们可以看到它是一个可执行协程的队列，可以无锁访问，因为本地队列就只有一个线程访问。当中 `runq` 是一个 256 长度的协程指针，并且使用 `runqhead` 以及 `runqtail` 来作为队列的头指针和尾指针。

接着是一个更重要的指针成员，名为 `runnext`, 指向下一个将要运行的协程。这个结构图简化成下面的图片:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/gx8D8iwOAThi20tywTD9.jpeg)

**p 从哪里获取协程呢？当然是从本地队列，如果本地队列没有了呢？就从全局队列中在拿去一批协程进入本地队列。那么如果全局队列中也没有了，但是在其他线程队列中还有没有执行的协程，那么就从其他队列“偷”协程进入本地队列执行。如果在这个过程中创建了一个新的协程，不会直接进入全局的队列，而是会找寻空闲的本地队列投放，以此来减少全局队列锁的竞争。**这些代码逻辑都在源码的 `runtime/proc.go` 文件的 `schedule` 方法里：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/Dl5yTWvyacunvBHSsXd7.png)

`gp` 这个变量在前文中以及说明，它是即将要执行的协程。它是怎么获取的呢？看第 3182 行代码，`runqget(_g_.m.p.ptr())`，解释一下这个入参，`_g_`这个指的是当前的协程，`_g_.m` 指的是当前的线程，所以连起来 `_g_.m.p.ptr()` 指的是当前协程所在的线程中的处理器的指针。然后我们来看这个方法的执行:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/PaEeA1PwiIg7We3OJAFx.png)

首先取出 `_p_.runnext`，它是下一个要执行的协程。如果它是存在的，就返回执行。如果没有取到呢？我们看上一张代码截图中的第 3187 行代码，`gp == nil`就 `findrunnable()`, 这个方法会从全局队列中找, 你可以看到第 2561 行 `globrunqget(_p_, 0)`：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/6lGpCUTdHdUSWRqn5NTp.png)

如果全局队列中也没有了呢？然后我们继续看 `runtime.schedule`方法, 在第 2599 行代码中:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/8rA2R2fIXKvoshV0UvCU.png)

`stealWork` 这个方法就是从其他 Processer 中“偷取”协程来执行，这样做的目的就是不让协程因为全局队列中没有就空闲着，也减少其他本地队列的库存，提升线程的总体利用率。

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/cc5mb8BIMHw5ZYItBFan.png)

总结一下 Processer 的作用：**它是 M 和 G 之间的中介， P 持有一些 G，使得每次获取 G 不需要从全局找从而大大减少了并发冲突。**

另外需要说明的是，如果是新建的协程，不会先进入全局队列，而是会直接加入到相对空闲的本地队列中去，并且优先执行新建的协程，就是在本地队列中仍然插队。这个源码在 `runtime/proc.go`中，方法 `newproc`：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/ljw97nGAJsj1secvEaIA.png)

“插队”的逻辑就在第 4059 行的 `runqput()`方法中。

### 协程饥饿问题 {id="hunger"}

到目前为止，我们每一个线程中的协程只能逐个执行。如果我们一共有 8 个线程，那么最多同时执行 8 个协程。这时候就有可能会产生协程的饥饿问题，比如某一个协程的执行特别耗时间:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/6yJS8mWfbH59XE0yzwrk.jpeg)

那么问题是提出来了，如何解决这个问题呢？思路是让这类需要长时间执行的协程暂停一下，也给其他的协程执行的机会。如何暂停呢？就是将协程的执行进度、状态都保存在协程结构体(g)中，并将它放回全局队列或者其他地方，然后跳转到线程循环的开始 `schedule()` 方法，调度队列中等待的协程进场执行。如下图所示：

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/IHYDlz3e4KIeQFboZmrq.jpeg)

这样就没有问题了吗？问题总是会有的，本地队列中的协程现在循环执行了，可是全局队列中的协程呢？在本地队列中的协程执行都非常 耗时的情况下，它们也是协程，它们也会感到饥饿的。

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/EbG1K3MDARdx9I0asABJ.jpeg)

那么在 Go 语言中是如何解决这个问题的呢？我们可以在 `runtime/proc.go` 的 `schedule()` 方法中，看到处理方法。就是当执行了 61 次本地队列中的协程之后，就从全局队列中取出 1 个协程加入到本地队列中，参与执行循环。这样也解决了全局队列的饥饿问题:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/3xc2Vt2l5oZoK74Lh2i0.png)

可以在第 3175 行中看到它对 61 这个数字取模，然后再第 3177 行看到，每次从全局队列中取 1 个（max 为 1）。

### 协程的切换时机 {id="switching"}

解决协程饥饿问题的关键在于，在协程执行到一半的时候按下暂停键。那么这是怎么做到的？在什么时候协程会暂停，然后保存线程让其他协程入场执行呢？一般来说，有两种做法：

- 主动挂起：当我们的业务协程主动调用了 `runtime.gopark` 方法，就会暂停执行，然后重新开始线程循环(跳转到 `schedule()` 方法取执行）。
- 系统调用完成时: 当协程在完成了系统调用后，暂停执行

### 主动调用 gopark 挂起 {id="call-gopark-hang-up"}

我们先来看`runtime.gopark()` 方法， 代码如下：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/dFPILktrfg1KcRyiS6Ef.png)

这个方法的主要逻辑是维护的协程的一些状态， 比如等待的原因之类的。最后调用了 `mcall()`方法，之前有说到过这个方法会切换协程通用栈（g stack） 到 g0 stack。而 `park_m` 这个方法如下:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/OQW3JiAImJZENYeHvZzy.png)

你可以看到在第 3336 行调用了 `schedule()` 方法，重新回到了线程循环的开始。

但是需要注意的是，`gopark` 这个方法在我们的业务代码中是无法使用的，因为方法名开头是小写啊。虽然如此，但是我们在业务方法中通过 `time.Sleep()` 之类的方法暂停协程，实际上在底层调用的都是 `gopark()`。

另外，`gopack` 方法调用之后是不会理解进入我们的协程本地队列的，而是会进入等待状态。这一点我们可以通过 `gopark`方法的注释看到:
> // Puts the current goroutine into a waiting state and calls unlockf on the system stack.  —— `runtime/proc.go` 第 327 行注释。


### 系统调用完成时 {id="when-system-called"}

第二种方法就是在程序完成了操作系统提供的系统调用 （System Call) 后，会调用 `runtime/proc.go` 文件中的 `exitsysyscall`方法，方法代码如下：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/jjyK4sOHrj0RHDM6pjJK.png)

然后这个方法会在第 3782 行调用 `Gosched()` 方法：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/AZGqgjUbm2B8jgutukL3.png)

`Gosched()`这个方法你看调用了 `mcall` 来切换 gstack 到 g0 stack，执行了 `gosched_m` 方法， 如下：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/U7DYO1Z0P9NLFaEqe06E.png)

接着在第 3359 行调用了 `goschedImpl`方法，代码如下：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/jMNNYNhAsfdUKb2AQ55z.png)

最后在第 3351 行调用了 `schedule` 方法，回到了线程循环的最开始。但是到目前为止，这还不能说是一个完善的解决方案，至少还存在一个问题：如果一个协程永远没有主动调用 `gopark` 呢？并且也没有调用任何系统调用呢？是不是就不会参与到协程调度？

### 编译器插入 morestack 方法 {id="insert-morestack-method"}

实际上，如果我们将最开始的示例程序编译输出汇编的话，会发现在汇编中调用了一个名为 `morestack` 的方法，我们输入如下命令来输出汇编：

```shell
$ go build -gcflags -S main.go
...省略前面输出内容...
0x0052 00082 (main.go:10)       MOVQ    56(SP), BP
0x0057 00087 (main.go:10)       ADDQ    $64, SP
0x005b 00091 (main.go:10)       RET
0x005c 00092 (main.go:10)       NOP
0x005c 00092 (main.go:8)        PCDATA  $1, $-1
0x005c 00092 (main.go:8)        PCDATA  $0, $-2
0x005c 00092 (main.go:8)        NOP
0x0060 00096 (main.go:8)        CALL    runtime.morestack_noctxt(SB)
0x0065 00101 (main.go:8)        PCDATA  $0, $-1
0x0065 00101 (main.go:8)        JMP     0
...省略后面输出内容...
```

当中关键的内容是`CALL runtime.morestack_noctxt(SB)` 这句，这并不是我们自己代码中调用的，而是在编译阶段由编译器插入的，当我们在程序中调用了
其他方法的时候。它的作用就是检查协程栈(g stack)是否拥有足够的空间来支持方法调用，**它会检查我们的协程运行是否超过 10ms, 如果是则标记为抢占，将
**`**g.stackguard0**`**这个标志设置为 **`**0xfffffade**`。如果是被抢占了，直接回调线程循环的 `schedule()` 方法。我们透过源码来看：

在 `runtime/stub.go` 中，有两个桩方法：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/TfVPODfkC0NLwTNJcpiW.png)

所以这两个方法都是使用汇编实现的，我们先来看 amd64 的 `morestack_noctxt` 的实现:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/tQMlFYCeFRqXRPWIKYId.png)

实际上只是跳转到了 `runtime.morestack`, 区别是并不保存上下文。接着我们来看 `morestack` 的实现:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/TKzhHbqv4tLCWzIj2qW1.png)

这个方法就在 `runtime.morestack_noctxt` 方法的上面，前面的逻辑都是处理栈空间不足，我们主要关注第 548 行代码, 调用了 `runtime.newstack` 方法(位于 `runtime/stack.go`)，这个方法主要是生成一个新的栈，但关键是下面的代码：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/jHQ6D6oQEV59tRMwIvdy.png)

`preempt` 这个单词就是抢占的意思，对比 `stackguard0`是否为 `stackPreempt`, `stackPreempt`的值卸载了下面的注释中, 就是 `0xfffffade`:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/sMAkhVQ9JUBzOZyiIxJy.png)

如果是抢占怎么办呢？我们继续看源码:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/8W8248R3kXJjYKmu0YEh.png)

调用了 `gopreempt_m` 方法，这个方法接着调用了`goschedImpl` 方法：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/G0lO2lPp5ADoSsYc3ntg.png)

关键在于这个方法的最后一行，调用了 `schedule()` 方法，回到了线程循环之初:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/2yxsuFyQxnQOh4ul8PDy.png)

到这里为止我们就引入了第三个协程切换的方案(时机), 在一个程序中方法调用是非常频繁的，所以在方法调用的时候通过 `morestack` 方法的插入，来标记协程是否超过 10ms, 如果是则标记为抢占，并且调用 `schedule()`。但是如果我们的协程中没有调用其他任何方法呢？比如下面的这个示例:
```go
func forever() {
	i := 0
	for true {
		i++
	}
}
```
### 基于信号的抢占式调度 {id="signal"}

上一小节中我们提到，如果协程中是一个死循环，即没有主动调用 `gopark` 也没有发生任何的系统调用，甚至没有调用任何的方法，那么协程就没有机会发生调度。如果几个线程中都是这样的方法，那么队列中的其他线程就会被阻塞。因此，Go 又引入了基于信号的抢占式调度。

操作系统提供了很多的信号，可以通过发送信号来和线程进行通信。线程会实现注册信号的处理函数，当接受到信号之后就会调用处理函数去处理。

继续来看源码，在文件 `runtime/signal_unix.go` 中，有一个方法是 `doSigPreempt`， 处理操作系统发送的 `SIGURG` 信号：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/clI7FEMCVfuqpIGia6Lm.png)

我们看第 345 行，调用了 `asyncPreempt` 方法，这个方法是使用汇编实现的。最终会调用同文件中的 `asyncPreempt2` 方法：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/wgz9cTZg2VfvAOIO9tw9.png)

第 306 行，通过 `mcall` 方法切换协程栈，然后调用 `preemptPark` 方法：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/gkevPDlL9EmGfBGXi8mG.png)

你看这个方法的最后，还是调用了 `schedule()` 方法，回到线程循环的最初。

## 总结 {id="summary"}

Go 语言的并发模型基于 GMP（Goroutine-Monitor-Processor）架构，该模型是 Go 运行时为支持高效率、大规模并发而设计的核心机制。以下是 GMP 模型的关键组成部分及其工作原理：

1. **Goroutines (G)**:
    - Goroutines 是 Go 语言中的轻量级用户态线程，可以低成本地创建和销毁。
    - 每个 Goroutine 都有一个独立的栈空间，其大小可动态增长或收缩以适应执行需求。
    - Goroutine 包含了要执行的任务函数、状态信息以及上下文切换所需的结构。

2. **Machines (M)**:
    - M 表示内核线程抽象，代表操作系统线程。
    - M 负责在绑定到 P 后执行实际计算任务，并与系统调度器交互，实现真正的并行执行。
    - M 中并不直接保存 Goroutine 的上下文，这使得一个 Goroutine 可以在不同的 M 上运行。

3. **Processors (P)**:
    - P 代表逻辑处理器，是 Goroutine 执行的调度单位，类似于 CPU 核心的概念。
    - P 拥有自己的本地任务队列（长度通常为 256），用于存放待执行的 Goroutine。
    - P 提供了内存分配的状态和其他资源管理服务，它必须与一个 M 绑定才能执行任务。
    - M 和 P 的绑定关系是可以动态改变的，通过这种动态绑定，Go 运行时可以将多个 Goroutine 在有限数量的 M（即内核线程）之间进行高效调度，从而实现多路复用。

4. **调度过程**：
    - 当 Goroutine 创建后，会被加入全局队列或某个 P 的本地队列中。
    - 调度器根据可用的 P 将等待执行的 Goroutine 分配给 M 执行。
    - 如果某个 M 结束了当前 Goroutine 的执行，且其关联的 P 的本地队列为空，则可以从全局队列或者其他 M 的本地队列获取新的 Goroutine 来执行。
    - 用户可以通过 `runtime.GOMAXPROCS()` 或环境变量 `GOMAXPROCS` 来设置最多允许同时运行的 M 数量，从而控制程序的并行度。

GMP 模型使得 Go 程序能够充分利用多核硬件的优势，同时简化了并发编程模型，减少了程序员对底层线程管理的关注，使他们更专注于业务逻辑的实现。
