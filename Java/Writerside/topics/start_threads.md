# 实现线程的方法

在网络上，关于实现线程有几种方法众说纷纭，2种、3种、4种、6种的说法都有。到底是几种呢？应该说是两种。一种是将类声明为 Thread 的子类，一种是声明一个实现了 Runnable 接口的类，然后实现其 run 方法。这篇文档将详细描述这两种实现线程的方法。

## 实现 Runnable 接口 {id="runnable"}

这种方法要求我们的类实现 Runnable 接口以及其 run 方法。最简单的示例如下:
```Java
public class Task implements Runnable {
    @Override
    public void run() {
        System.out.println("Hello World!");
    }
    public static void main(String[] args) {
        Thread thread = new Thread(new Task());
        thread.start();
    }
}
```

## 继承 Thread 类

这种方法要求我们的类继承 `Thread` 类，并重写其 `run` 方法:
```Java
public class TaskStyle extends Thread {
    @Override
    public void run() {
        super.run();
        System.out.println("Hello World!");
    }
    public static void main(String[] args) {
        Thread thread = new Thread(new TaskStyle());
        thread.start();
    }
}
```

## 两种方式的对比 {id="diff_runner_and_thread"}

使用继承 Thread 这种方法存在 3 个缺点：

1. 从代码架构角度：具体的任务（run方法）应该和“创建和运行线程的机制（Thread类）”解耦，用runnable对象可以实现解耦。

2. 使用继承 Thread的方式的话，那么每次想新建一个任务，只能新建一个独立的线程，而这样做的损耗会比较大（比如重头开始创建一个线程、执行完毕以后再销毁等。如果线程的实际工作内容，也就是run()函数里只是简单的打印一行文字的话，那么可能线程的实际工作内容还不如损耗来的大）。如果使用Runnable和线程池，就可以大大减小这样的损耗。

3. 继承Thread类以后，由于Java语言不支持双继承，这样就无法再继承其他的类，限制了可扩展性。

所以我们会选择使用实现 Runnable 的方法。

通过 `Thread.java` 中 run 方法的源码，我们可以看到两种方式本质上是一样的，都是去调用我们写的 run 方法。只是接口的形式，是调用我们实现类中的 run 方法，而继承的方式是直接覆盖了 `Thread.run` 方法。源码如下:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/bd1rpnQJmKYiGZZDAdRH.png" alt="runner source" />

## 同时使用两种方式创建线程 {id="runner_and_thread"}

那么如果我们同时使用这两种方式去创建线程，会发生什么情况呢？是两个都运行、都不运行、或者其中一个运行呢？实验的代码如下:
```Java
new Thread(new Runnable() {
	@Override
	public void run() {
		System.out.println("Hello, Runnable!");
	}
}){
	@Override
	public void run() {
		// super.run();
		System.out.println("Hello, Thread!");
	}
}.start();
```

运行这行代码，最终会输出 Hello, Thread!。我们如果取消第 9 行的代码的注释，那么结果又会如何呢？两句输出都会执行。那么这是为什么呢？

这是因为，我们重写了`Thread.run()` 方法，所以不会再去执行父类中的代码, 即 `if(target != null) target.run()`，自然就不会去执行 Runnable 匿名类中的 `run`。但是如果我们取消第九行的注释，都可以。

## 其他方式 {id="other"}

有一种说法认为“通过线程池创建线程”也是一种方式，实际上本质还是通过实现 Runnable 接口来创建线程的。其他的说法，比如定时器、匿名类、Lambda 等方式都可以创建线程，但是本质上都是对这两种方式的包装，本质上都逃不脱这两种方式。

## 总结 {id="summer"}

实现线程有两种方式，分别是通过继承 Thread 类以及实现 Runnable 两种。但是由于继承 Thread 存在一些缺陷，所以我们会优先使用 Runnable的方式来实现线程。

但是究其本质，实际上创建线程只有一种方法，那就是构造 `Thread` 类，运行其 `run` 方法。只是上面两种方法，获取 `run` 方法的途径不一样。一种是通过继承覆写 `run` 方法，另一种是通过实现 `Runnable` 接口的 `run` 方法，并将实现类通过 `Thread` 类的构造方法传入并赋值给其 `target` 属性。

