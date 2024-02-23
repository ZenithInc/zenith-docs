# GIL

GIL(**G**lobal **I**nterpreter **L**ock) 是 Python 解释器(CPython) 中对于线程并发锁的实现，用来保护临界资源在并发下安全的措施。这篇文章简要介绍了它的历史。

## GIL {id="what-is-gil"}

GIL（Global Interpreter Lock）  是 Python 解释器(CPython) 中对于线程并发锁的实现，用来保护临界资源在并发下安全的措施。

临界资源指的是一些虽然作为共享资源却又无法同时被多个线程共同访问的共享资源。当有进程在使用临界资源时，其他进程必须根据操作系统的同步机制等待占用进程释放该共享资源才可重新竞争使用共享资源。

因为使用了GIL，所以解释器只允许同一时间执行一个线程。所以，在 Python 中多线程是并发而不是并行，并发是指在代码算法层面的分割，并行指的是多核下执行层面同时。也因为 GIL ，所以 Python 中的多线程是伪多线程，并不能利用到机器的多核资源。

## 为什么要保留 GIL 呢 {id="why-does-python-retain-the-gil"}

这是为了当我们多进程（多线程）中操作类似于 Map，Tuple、Dict 这样的数据结构容器的时候，需要使用 GIL 锁来保证这些结构是安全的。

比如 map.append这样的代码在 Python 中只有一行代码，而在底层执行的时候可能是多行代码。在多进程的场景下，如果没有锁的存在就会导致线程不安全。

在具体的实现上，Python 的作者并没有给每一种数据结构都加上锁。而是在全局层面加了锁，所以这就是被称为 GIL 的缘故。

这样简单的实现是有历史原因的。第一版本的 Python 是在 1989 年诞生的，那时候就有了 GIL，但那时候并没有多核 CPU(**第一颗双核 CPU 出现在 2005 年，Intel 奔腾系列**)。所以这样的实现在那时候并不成问题。

但是随着 Python 的后续发展，包括官方提供的标准库以及第三方实现的类库，大都依赖于这个 GIL ，所以这没有办法简单的去掉。

另外，如果将 GIL 这种全局锁去掉的话，就会导致在内部很多地方都要加上粒度更细的锁。这样虽然没有了全局锁，但是数量更多粒度更细的锁，会导致频繁地加锁解锁，从而导致 Python 的整体性能更差 (曾经有人这么做，并给出数据说在单线程下运行效率会降低 50%）。


## Python 历代版本中的 GIL {id="the-GIL-history"}

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/Rz049HcAETu7cXLBx9Hs.jpg" alt="The GIL History" />

## 总结 {id="summary"}

这篇文档介绍了Python解释器中的全局解释器锁（GIL）以及其历史和原因。以下是对文档的总结：

**1. 什么是 GIL：**
- GIL是Python解释器（CPython）中的全局解释器锁，用于保护临界资源在并发情况下的安全访问。
- 临界资源是指那些虽然是共享资源，但不能同时被多个线程访问的资源。

**2. GIL的作用：**
- GIL的存在使得解释器一次只允许执行一个线程，因此在Python中，多线程是并发而不是并行。
- 由于GIL的限制，Python中的多线程被称为伪多线程，无法充分利用多核处理器的资源。

**3. 为什么保留 GIL：**
- GIL的保留是为了在多线程/多进程操作中保证对类似Map、Tuple、Dict等数据结构容器的安全操作。
- 在多进程场景下，如果没有锁的存在，可能导致线程不安全，因此需要GIL来确保数据结构的安全性。

**4. GIL的历史和原因：**
- GIL最初出现于Python的早期版本（1989年），当时并没有多核CPU。因此，这样的实现在当时并不是问题。
- 随着Python的发展，包括标准库和第三方库在内的许多实现都依赖于GIL，因此去除GIL并非简单的任务。
- 去除GIL将导致在许多内部位置需要使用更细粒度的锁，这可能会导致性能下降。曾经有人试图去除GIL，但在单线程下运行效率降低了50%。

总体而言，GIL在Python中是为了简化多线程操作的实现而存在的，但它也带来了一些性能上的限制。因此，解决GIL的问题一直是Python社区关注的焦点之一。