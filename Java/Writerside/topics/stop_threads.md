# 停止中断线程

这篇文档我们对停止以及中断线程的最佳实践进行介绍，包括停止线程的原理，如何正确地停止线程，哪些方式停止线程是错误的，停止线程相关的重要函数。

这篇文档我们对停止以及中断线程的最佳实践进行介绍，包括停止线程的原理，如何正确地停止线程，哪些方式停止线程是错误的，停止线程相关的重要函数。

### 停止线程的最佳实践 {id="good_example"}

正确停止线程的方式是**使用**`**interrupt**`**来通知，而不是强制停止**。Java 语言没有提供一种安全可靠的方式来停止线程，但是提供了`interrupt`, 它是使用一个线程来通知另一个线程停止。

`interrupt`是中断，是一种协作机制，而不是停止。使用它通知线程要停止，但是要不要停止、何时停止，这都是由线程自己决定的，如果线程不愿意停止，那么我们也无能为力。所以说，是通知，而不是强制停止。**这依赖于请求停止方和被停止方都遵守一种约定好的编码规范。**

如果是立即停止，会导致正在共享的数据结构处于不一致的状态。如果是通知的话，正在运行的线程可以做一些善后工作。因为线程本身更清楚自己什么时候要停止，应该如何停止。

### 如何停止线程 {id="how"}

线程会在两种情况下停止，一种是在 `run` 方法执行完毕之后，第二种是运行过程中出现了异常，且没有捕获处理。还有就是线程被阻塞了，或者是在每次循环中都阻塞。

下面的代码演示了在`run` 方法中不存在任何阻塞的情况下，使用 `interrupt`:

```java
Runnable runnable = () -> {
    for (int i = 0; i < Integer.MAX_VALUE / 2; i++) {
        if (Thread.currentThread().isInterrupted()) 
            break;
        if (i % 10000 == 0) {
            System.out.println(i);
    }
};
Thread t = new Thread(runnable);
t.start();
Thread.sleep(1000);
t.interrupt();
```

主要是第 3 行，我们通过`isInterrupted`方法来检查是否收到中断信号，如果是则退出。还有是第 11 行和第 12 行，我们休眠了 1 秒之后通过主线程向任务线程发送了中断信号。

### 如果遇到阻塞怎么办 {id="interrupt"}

接着，我们来看看如果`run`方法中存在阻塞怎么办，下面是代码示例:

```java
Runnable runnable = () -> {
	System.out.println("start...");
	try {
		Thread.sleep(1000);
		System.out.println("end...");
	} catch (InterruptedException e) {
		System.out.println("Interrupted during sleep");
	}
};
Thread t = new Thread(runnable);
t.start();
Thread.sleep(500);
t.interrupt();
```

如果存在阻塞，只要捕获`InterruptedException` 异常并正确处理即可。如果是在循环中，每次迭代都会阻塞的情况呢？一样是通过捕获异常，只是不需要每次都通过`isInterrupted`方法去检测了。

### 在 while 中捕获中断的问题 {id="while"}

如果我们在 `while`这样的循环中捕获异常，就会出现不一样的情况:

```java
Runnable runnable = () -> {
	int count = 0;
	while(!Thread.currentThread().isInterrupted() && count < 100) {
    	try {
    		System.out.println("count++ = " + ++count);
    		Thread.sleep(10);
    	} catch (InterruptedException e) {
    		System.out.println("e.getMessage() = " + e.getMessage());
    	}
	}
};
Thread t = new Thread(runnable);
t.start();
Thread.sleep(500);
t.interrupt();
```

一个是当我们处理了异常之后，循环仍然会继续。然后我们在第 3 行调用`isInterrupted`方法进行判断，如果存在中断信号则退出循环，这个判断是不生效的。不生效的原因是，当我们在第 8 行已经处理了中断信号之后，Java 就会清除中断标记，在第 3 行的判断就会不生效。

在编写`run` 方法的时候，要注意，如果是调用了其他方法，其他方法里最好不要处理中断，而是将中断传递给顶层方法(`run`)来处理。如果一定要处理，也可以，不过要恢复中断:

```java
try {
	// 具体的业务逻辑
} catch (InterruptedException e) {
    // 再次调用中断, 恢复中断
    Thread.getCurrentThread().interrupt();
	// 做自己想做的事情
}
```

所以，**切记不要屏蔽中断**。

### 可以响应中断的方法 {id = "interrupted_response"}

下面这些方法可以响应中断, 调用下面这些方法，如果发生中断，我们的程序是可以感知到的:

- `Object.wait()`
- `Thread.sleep()`
- `Thread.join()`
- `java.util.concurrent.BlockingQueue.take() / put(E)`
- `java.util.concurrent.locks.Lock.lockInterruptibly()`
- `java.util.concurrent.CountDownLatch.await()`
- `java.util.concurrent.CyclicBarrier.await()`
- `java.util.concurrent.Exchanger.exchange(V)`
- `java.nio.channels.InterruptibleChannel` 相关方法
- `java.nio.channels.Selector` 相关方法

### 错误停止线程的方法 {id="error_example"}

第一种是通过 `stop`、`suspend`和 `resume` 方法。这三个方法目前都已经被弃用了，所以使用的时候 IDE 也会给出提示，不要依赖。

还有一种方法，采用`volatile` 关键字修饰成员标量，通过更改成员变量的标志来停止线程。下面代码演示:

```java
public class Task {
	// 设置通过 volatile 修饰的标志位
    // 这个变量就可以在多个线程之间可见
    protected volatile static boolean canceled = false;

    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = () -> {
            int count = 0;
            while(!canceled && count < 100) {
                try {
                    System.out.println("count++ = " + ++count);
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    System.out.println("e.getMessage() = " + e.getMessage());
                }
            }
        };
        Thread t = new Thread(runnable);
        t.start();
        Thread.sleep(300);
        canceled = true;		// 更改标志位
    }
}
```

上面的代码是正常可以运行的。但是并不是说这种方法就是对的，有的情况则不行。就看上面的代码，如果线程长时间阻塞，就根本走不到第 9 行去检测 `canceled` 变量的值。这可以参考《Java并发编程实战》一书。