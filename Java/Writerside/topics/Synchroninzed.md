# Synchroninzed

这篇文档将对 Synchronized 做深入的解析。它是 Java 的关键字，被 Java 原生支持，是最基本的互斥同步手段。

### 什么是 Synchronized {id="what"}

官方对 Synchronized 关键字的解释，大意是同步方法支持一种简单的策略来防止线程干扰和内存的一致性错误：如果一个对象对多个线程可见，则对该对象变量的所有读取或写入都是通过同步方法完成的。原文如下:

> Synchronized methods enable a simple strategy for preventing thread interference and memory consistency errors: if an object is visible to more than one thread, all reads or writes to that object's variables are done through synchronized methods.


简单来说， Synchronized 关键字修饰后，能够保证在同一时刻最多只有一个线程执行该段代码，以达到保证并发安全的效果。实际上，它是通过独占锁实现的。

### 一个线程不安全的示例 {id="unsafe"}

我们先写一个线程不安全的示例，通过计数来掩饰:

```java
public class Application implements Runnable {

    static int i = 0;

    static Application application = new Application();

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(application);
        Thread thread2 = new Thread(application);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("i = " + i);
    }

    @Override
    public void run() {
        for (int j = 0; j < 10000; j++) {
            i++;
        }
    }
}
```

按理来说，两个线程对同一个变量计数，每个 10 万次，那么最终的结果应该是 20 万。实际上结果是不一定的。可能是十几万吧。这是因为 `i++`这个操作并不是原子性的。

### Synchronized 的两个用法 {id="usage"}

这个关键字有两个用法，如下:

- **对象锁**: 包括方法锁（默认锁对象为 this 当前实例对象）和同步代码块锁（自己指定锁对象）
- **类锁**：指 synchronized 修饰静态的方法或指定锁为 Class 对象

#### 对象锁 {id="object_lock"}

还是以上面的例子来说，改写 `i++` 这句代码如下:

```java
for (int j = 0; j < 10000; j++) {
	// i++ 替换如下
	synchronized (this) {
		i++;
	}
}
```

我们也可以将 `this` 改为自己指定的对象:

```java
final static Object lock = new Object();
synchronized (lock) {
	i++;
}
```

为什么要自定义对象呢？因为在实际过程中，一段代码中可能会有好几把锁，我们可以定义 `lock1` 、`lock2`等等。

#### 类锁 {id="class_lock"}

所谓的类锁，只不过是 Class 对象的锁而已。类锁指的是在同一时刻被同一个对象拥有。我们来看下面这个示例:

```java
public class Application implements Runnable {

    static int i = 0;
	// 两个实例
    private static final Application application1 = new Application();
    private static final Application application2 = new Application();

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(application1);
        Thread thread2 = new Thread(application2);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("i = " + i);
    }

    @Override
    public void run() {
        for (int j = 0; j < 10000; j++) {
           incr();
        }
    }
	// 非 static 修饰
    private synchronized void incr() {
        i++;
    }
}
```

因为`application1`和`application2`是两个不同的实例，所以他们获取到的锁也是不同的锁。所以这个程序也是线程不安全的。要让这个程序线程安全，只需要在 `incr`方法上修饰 `synchronized`就可以了。

还有第二种形式，我们使用类名作为锁：

```java
@Override
public void run() {
	for (int j = 0; j < 10000; j++) {
		synchronized (Application.class) {
			i++;
		}
	}
}
```

虽然衍生出来的实例是不同的，但是都是这个`Application`的类的实例，所以依然能排队运行。