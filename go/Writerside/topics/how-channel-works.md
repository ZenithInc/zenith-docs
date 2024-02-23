# Channel 原理解析

在 Go 语言中，Channel（通道）作为一种内建的通信原语，扮演着并发编程的核心角色。它为协程间的同步和数据交换提供了一种直接、高效且易于理解的方式，
极大地简化了并行任务的设计与实现。Channel 是一种类型化的管道，允许我们发送和接收特定类型的值，并通过其独特的设计保证了操作的有序性和一致性。

## 使用 Channel 而不是共享内存  {id="use-channel"}

Go 官方推荐使用通信的方式来共享内存，而不是通过共享内存的方式进行通信。这句话怎么理解呢？我们先来看一个通过共享内存的方式来通信的例子:
```go
func doPrint(p *int) {		// 传递的 p 是一个指针，是一块内存地址
	for {				// 不断循环检测 p 指针的值是否为 1
		if *p == 1 {
			fmt.Println("Hello")
			break
		}
	}
}
func main() {
	num := 0
	go doPrint(&num)	// 通过传递内存地址来通信
	time.Sleep(time.Second)
	num = 1			// 当 num = 1 的时候，打印 Hello
	time.Sleep(time.Second)
}
```
通过这个例子，我们可以看到它是通过传递地址的方式去通信的，协程`doPrint` 需要采用死循环不断地检测 `p` 指针的值是否为 1。

同样的例子，我们使用 Channel 来重写这个例子, 然后来对比一下:
```go
func doPrint(c chan int) {
    // 取消了 for 循环
	if <-c == 1 {				// 通过 Channel 接收数据
		fmt.Println("Hello")
	}
}
func main() {
	c := make(chan int)		// 初始化无缓存 Channel 类型为 int
	go doPrint(c)
	time.Sleep(time.Second)
	c <- 1					// 发送数据
	time.Sleep(time.Second)
}
```
通过对比，Channel 通信相比共享内存（传递指针）的好处如下:

- 避免协程竞争和数据冲突的问题
- 更高级的抽象，语义更加明确，降低了开发难度，增加程序的可读性
- 模块之间更容易解耦，增强扩展性和可维护性
- 
## Channel 的底层结构 {id="struct"}

在 Go 源码的`chan.go` 文件中，有 Channel 的类型`hchan`，其源码如下:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/rxp4tDyRxuh003sIITxJ.jpeg)

## Channel 底层原理 {id="how-it-works"}

在了解了 Channel 的底层数据结构之后，我们接着来看当我们发送数据或者接受数据的时候，Channel 在底层都做了些什么。

### 发送数据 {id="send-data"}

我们使用 `c<-` 这样的语法来发送数据，`<-` 这只是一个语法糖，编译器会在编译阶段，将其转换为 `runtime.chansend1()` 这个方法。在 `chan.go` 这个源码文件中，就有这样一个方法:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/dKKZ26OKSikEcAjNLWIm.png)

往 Channel 中发送数据分为三种情况，如下图所示:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/UAmG1mT9Ytinnhe3feMM.jpeg)

#### 直接发送 {id="direct-send"}

我们来看`chansend`这个方法的源码, 核心代码如下：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/0NDwxfHxJfvgJD8h2iIc.png)

第 201 行，因为要发送数据，所以先加 Mutex 锁。第 203 行，检查 Channel 是否 Closed。第 208 行，从接收队列中取出一个协程，然后第 211 行，`send` 发送数据。

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/yWjtRS3Ajv9Qe5dlGEFg.png)

第 310 行，判断 `sg.elem`不为空，这个指的是接收数据的协程中，那么接收 Channel 数据的变量，比如说`i <- chan`, 那么就是 `i` 这个变量。然后 `sendDirect` 直接发送:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/JhzlMxHGXLqEwPBahWKD.png)

第 344 行，`memmove`这个方法就是将数据直接拷贝到 `sg.elem` 中去。最后唤醒协程:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/qcjl5oPNXHvraOoSUWTr.png)

第 321 行，`goready`方法唤醒协程。

#### 放入缓存区 {id="write-cache"}

上文我们已经解释了直接发送的源码，下图中的第 208 行，从休眠的等待队列中取出一个协程，如果协程不存在，然后就在第 215 行判断 Ring Buffer 是否已经满了:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/2vGe7X4buc80kLI5BMu1.png)

第 217 行，从 Ring Buffer 中取出可用的一个缓存单元，然后第 221 行，将我们缓存的数据通过 `typedmemmove`方法移动到缓存单元。最后维护 Ring Buffer 的索引, 第 223 行到第 226 行。

这种方式不需要和其他任何协程打交道，只需要判断 Ring Buffer 缓存是否有空，将数据存入即可。**所以，Channel 不是无锁的，只是加锁的时间非常短而已**。

#### 休眠等待 {id="sleep"}

如果 Ring Buffer 已经满了，或者在 Receive Queue 中没有等待接收数据的协程，那么协程自己就会进入休眠等待，将自己包装成 `sudog` 结构，加入到 `sendq`队列中，并休眠等待解锁。

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/Dz8pJ4YQe5YDx9ve5rbY.png)

看截图中的代码:

- 第 237 行：获取自己的协程结构体
- 第 238 行：通过 `acquireSudog` 方法，将自己包装成 `sudog`结构体
- 第 245 行：把要发送的数据`ep`保存在 `sudog.elem` 成员中
- 第 247 行：自己协程的指针也要记录到 `sudog.g` 成员中
- 第 252 行：通过 `c.sendq.enqueue` 将自己加入休眠队列
- 第 258 行：通过 `gopark` 协程挂起休眠
- 
### 接收数据 {id="receive-data"}

我们使用 `<-c` 这样的语法来发送数据，`<-` 这只是一个语法糖，编译器会在编译阶段，根据接收形式的不同分为两种情况进行编译转换:

- 如果使用的是 `i <- c` 这样的语法，则转换为 `runtime.chanrecv1()`
- 如果使用的是 `i, ok <- c` 这样的语法，则转换为 `runtime.chanrecv2()` 方法

不管是 `runtime.chanrecv1` 还是 `runtime.chanrecv2` 方法，其实最后都会调用 `chanrecv()` 方法：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/nOy9Y4wySWwSCJr38iEM.png)

Channel 接收数据有四种情况：

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/PVEGwqNnAIIiJY9LQNA9.jpeg)

#### 已有协程在等待发送 {id="waiting"}

在这种情况下，Channel 是无缓存的或者缓存已满，而发送协程队列中有协程在休眠等待发送。我们来看 `chanrecv` 方法的源码:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/qGcSxVODdh5qR7BXNMAi.png)

下面截图中的第 522 行，从发送队列中获取一个休眠等待的协程，然后调用 `recv` 方法接收数据:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/aMvD7KZ3LYk2dmHQcj0J.png)

然后 `recv` 这个方法又分为两种情况，第一种情况如下图, 通过 `c.dataqsiz` 来判断缓存区是否是空的：

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/66JNgc2TfQHHTqWeGdTp.png)

如果是空的，则调用 `recvDirect` 方法，将数据直接拷贝，通过 `memmove` 方法:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/99RiaMzRyPamcuPnM4fI.png)

然后维护自己的状态，调用 `goready`唤醒协程。

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/mrhIvHac001OUbELD7qt.png)

其他三种情况也都类似，自行看源码。

## 实现非阻塞接受数据 {id="non-blocking-receive-data"}

当我们使用 `<-` 关键字来接受数据的时候，都是阻塞模式的。这是再源码中写死的，下图所示:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/aMwk26J7MmlFOpLNC6OF.png)

第 455 行，`chanrecv` 方法的第三个参数 `block` 是否阻塞，但是调用这个方法的两个方法 `chanrecv1` 和 `chanrecv2` 都写死为 `true`。那么如何做到非阻塞接受数据呢？使用 `select`。
```go
select {
    case <- chan1:
        // 如果 chan1 成功读取到数据，则执行该 case 语句
    case chan2 <- 1:
        // 如果 chan2 成功写入数据，则执行该 case 语句
    default:
        // 如果上面两个条件都不满足，则执行 default 语句
}
```

## 总结 {id="summary"}

本文深入剖析了 Go 语言中 Channel 的工作原理与实现机制，强调了使用 Channel 而非共享内存进行并发通信的优势。首先通过对比实例说明了 Channel 
在解决协程间同步和数据交换时的简洁性和安全性，以及其如何避免竞态条件、增强代码可读性和模块解耦性。

接着详细解析了 Channel 底层结构 `hchan`，并围绕发送和接收数据的过程，分析了 Channel 在不同情况下的具体操作步骤。在发送数据时，Channel 会
根据是否有待接收协程、缓存区是否已满等条件采取直接发送、放入缓存区或让发送协程进入休眠等待状态的不同策略。同时指出，尽管 Channel 不是完全无锁
的，但其加锁时间非常短，能有效降低锁竞争的影响。

在接收数据方面，文档详细阐述了四种不同的处理情况，并展示了如何通过源码理解 Channel 如何从发送队列获取数据并将其传递给接收者。此外，还介绍了 
Channel 的阻塞接收特性和如何通过 `select` 语句实现非阻塞接收。

总之，本文通过详细的源码解读和示例演示，全方位揭示了 Go 语言 Channel 的工作原理及其在并发编程中的高效应用，为开发者提供了深入了解和掌握这一重
要特性的重要参考。