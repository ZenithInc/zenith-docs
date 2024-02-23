# 线程的状态

线程的生命周期中一共有 6 个状态，这篇文档将会深入这些状态。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/dvm00khs7K46gfZkoqaP.jpeg" alt="status" />

关于阻塞状态需要特殊说明:

- 一般习惯吧 Blocked(被阻塞)、Watting(等待)、Timed_waiting(计时等待) 都成为阻塞状态
- 不仅仅是 Blocked 状态

下图展示了几种状态的转换:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/XYckWixSH7px3SYjdUNu.jpeg" alt="transfer" />


#### New

首先我们来使用代码演示 `NEW` 的状态。实现一个线程：

```java
public class NewRunnableTerminated implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            System.out.println("i = " + i);
        }
    }
}
```

然后创建线程:

```java
Thread thread = new Thread(new NewRunnableTerminated());
System.out.println("thread.getState() = " + thread.getState()); // NEW
```

#### Runnable

然后我们运行上面创建的这个线程:

```java
Thread thread = new Thread(new NewRunnableTerminated());
thread.start();
System.out.println("thread.getState() = " + thread.getState()); // RUNNABLE
```

#### Terminated

当线程执行完毕之后，就会进入到这个状态:

```java
Thread thread = new Thread(new NewRunnableTerminated());
thread.start();
Thread.sleep(10);
System.out.println("thread.getState() = " + thread.getState());
```

#### Time Waiting

实现一个线程类，在`run`方法中，`sleep`一秒钟:

```java
public class NewRunnableTerminated implements Runnable {
    @Override
    public void run() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

然后运行这个线程, 打印状态前先休眠 10 毫秒:

```java
Thread thread = new Thread(new NewRunnableTerminated());
thread.start();
Thread.sleep(10);
System.out.println("thread.getState() = " + thread.getState());
```

#### Waiting

比如我们调用了`Object.wait()`方法就会进入这个状态:

```java
public class NewRunnableTerminated implements Runnable {
    @Override
    public void run() {
        try {
            syn();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    private synchronized void syn() throws InterruptedException {
        Thread.sleep(1000);
        wait();
    }
}
```
测试如下:
```java
Thread thread1 = new Thread(new NewRunnableTerminated());
thread1.start();
Thread.sleep(1000);
System.out.println("thread1.getState() = " + thread1.getState());
```