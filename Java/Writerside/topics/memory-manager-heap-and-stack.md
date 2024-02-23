# 内存管理之堆栈

这篇文档描述了 JVM 中的堆栈结构，这里我们说堆栈实际上指的的栈,它是用来实现方法的调用和本地变量的存储的。之所以有时候我们说堆栈，是因为早些年从英文翻译过来并不准确。

## 数据结构中的栈 {id="data-structure-heap-and-stack"}

堆栈是两种在计算机科学中非常重要的数据结构。其中，堆是一颗完全二叉树，又分为最大堆和最小堆。如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/NrVIIyea60pWZslFBF7i.png" alt="max-heap-and-min-heap"/>

> 但是一颗二叉树又不一定是堆，因为不管是最大堆还是最小堆，都需要又一定顺序性。比如最大堆，每个节点的值必须大于子节点的值。而一般二叉树，对顺序并无特殊要求。

而栈呢？是后进先出（Last In,First Out, LIFO) 的线性表，要求插入（成为压栈）和删除（成为弹栈）都必须在一个位置上，我们称之为栈顶。如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/IP08VMg0A5ayDwbUO3UP.png" alt="stack"/>

## Java Virtual Machine 中的栈 {id="the-java-virtual-machine-heap-and-stack"}

在 Java 虚拟机中，堆和栈指的是堆内存进行操作和管理的方式，这个和数据结构中的堆栈不同。

**虚拟机栈的基本作用是方法的调用和局部变量的存储**，其结构如下：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/QHZFtSilyFAKUt1eRHwe.png" alt="the java virtual machine stack structure"/>

每一个 Java 的线程都维护了一个私有的栈，用来保存 Java 代码中的方法以及变量。除此之外还有一个 Native Stack，用来执行非 Java 编写的 C/C++ 的代码。

每一个栈帧 (Frame) 包含一下这些内容:

* **返回值**：当一个方法有返回值的时候，将返回值存储在这个位置，直到返回给调用方。
* **当前类常量池的引用**：每个栈帧都会包含一个对当前类常量池的引用。类常量池包含了编译时生成的各种字面量和符号引用，它是方法区的一部分。栈帧通过这个引用来访问常量池中的各种常量。
* **本地变量表**： 本地变量表是栈帧中的一部分，用于存储方法的局部变量。局部变量是在方法中定义的，其作用范围仅限于方法的执行过程中。本地变量表中存储了方法的参数和方法内部定义的局部变量。
* **操作数栈**： 操作数栈用于存储方法执行过程中的操作数。方法中的指令将操作数推入和弹出操作数栈，执行计算和逻辑运算等操作。操作数栈的深度在编译时就已确定。比如一个 `add(Integer a, Integer b)` 的方法中， `a` 和 `b` 就是两个操作数。



## 相关文章 {id="related-articles"}

1. [《JVM Internals》](https://blog.jamesdbloom.com/JVMInternals.html)