# 启动线程

上一篇文档我们讲了实现线程的两种方式，分为为继承Thread 类和实现 Runable类，并且我们推荐使用后者并分析了原因。这一篇文档我们详细描述了启动线程的两种方式，分别为调用start() 方法和run() 方法。

## 启动线程的两种方式 {id="the_way_of_start_thread"}

我们先写一个简单的示例, 通过两种方式去创建线程:
```Java
public class Task {

    public static void main(String[] args) {
        Runnable runnable = () -> {
            System.out.println(Thread.currentThread().getName());
        };
        runnable.run();      // 输出 main
        new Thread(runnable).start(); // 输出 Thread-0
    }
}
```
运行代码，可以看到`runable.run()` 输出的是`main`, 说明这个线程是通过主线程去执行的。而我们希望的是新建一个线程去执行，所以使用 `start()` 方法更好。

## start 方法原理分析 {id="start_source"}

`start()` 方法最主要的作用就是通知 JVM 虚拟机，在空闲的时候启动新线程执行任务。 所以线程什么时候执行，不是我们的程序决定的，而是线程调度器决定的，如果线程调度器比较忙碌，就可能会延迟执行。

另外需要明确的是，当我们调用了`ru`nnable.start()` 方法时，这句代码是我们的主线程执行的，就是 Main线程执行的，然后才是由线程调度器去执行具体的任务。

我们的任务线程也不是直接就能执行的，而是需要一些准备工作的。首先线程需要处于就绪状态，指的是获取到了除了CPU以外的其他资源，设置了上下文、栈以及PC寄存器(指向程序运行的位置）。  这些都完成了之后，才会会 JVM 调度到执行状态。

另外需要注意的是，不能重复执行 `start`方法。如下示例:
```Java
Runnable runnable = () -> {
    System.out.println(Thread.currentThread().getName());
};
Thread t = new Thread(runnable);
t.start();
t.start(); 
```
抛出`java.lang.IllegalThreadStateException`异常。字面上指的是线程状态错了，这是因为start 之后，线程状态已经发生了改变，而且不可逆，所以再次`start`就会报错。

首先，这个方法会去**检查线程状态**，我们通过源码可以看到:
```Java
public synchronized void start() {
    if (threadStatus != 0)
        throw new IllegalThreadStateException();
```
我们可以看到这个方法是被`synchronized`关键字修饰的，表示保证线程安全。另外，由 JVM 创建的 `main` 方法线程和 `system` 组里的线程，并不会通过 start方法来启动。

这个 0 是什么？我们可以查看`threadStatus`这个成员变量的定义, 默认就是零，注释也说清楚了表示 not yet started 还没有启动:
```Java
/*
* Java thread status for tools, default indicates thread 'not yet started'
*/
private volatile int threadStatus;
```
然后加入线程组:
```Java
group.add(this);
```
最后调用start0() 方法:
```Java
boolean started = false;
try {
	start0();
	started = true;
} finally { ...省略... }
```
我们来看 start0 方法的定义:
```Java
private native void start0();
```
这个方法的声明中有一个 native关键字，这表示这个方法并不是由 Java 实现的，而是通过 C /C++ 实现的。

## run 方法原理解析

`run` 方法的源码只有三行，如下:
```Java
@Override
public void run() {
	if (target != null) {
		target.run();
	}
}
```
所以`run`方法就是一个普通的方法，和我们在主线程中自己调用一个方法是一样的。所以，要启动一个线程，要调用`start` 方法。