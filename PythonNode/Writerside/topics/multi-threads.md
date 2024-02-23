# 多线程

当我们的程序被加载到内存中，是以进程的形式表现的。但是真正去执行我们程序的是线程，每一个进程都会有一个主线程。所以线程是属于进程的一部分。

进程的上下文切换对资源的花销是很大的，而线程又被称之为是轻量级的进程，所以相对进程的花销是小的。

### 获取线程 ID

当你你执行一个简单的脚本输出 `Hello World` 。实际上就是运行了一个进程，而这个进程中已经包含了一个线程，代码就是在线程中运行。

```python
import threading

def print_hi():
    print('Hi, {}'.format(threading.get_native_id()))

if __name__ == '__main__':
    print_hi()
```

> 注意: `get_native_id` 这个函数需要在 3.8 及以上版本才能使用.....

> 注意: 在 Linux 中，进程的 ID 和主线程的 ID 是一样的。可以使用 `ps -T` 命令来观察这一现象。



### 创建线程

在 Python 中如何创建一个线程呢？非常容易，使用一个 `threading.Thread` 类即可，这个类的原型如下:

```python
class threading.Thread(group=None, target=None, name=None, args=(), kwargs={})
```

这些形参的含义如下:

- group:   这是 group 的值，应该为 None;这是一个保留参数，供未来实现所用。
- target:   这是一个在启动一个线程活动时会执行的函数。
- name: 线程的名称，如果没有给定会以 `Thread-N` 的形式命名。
- args: 这是传递给 target 函数的参数。
- kwargs: 这是传递给 target 函数的关键字参数

```python
import threading

def run(thread_id):
    print("function called by thread %i\n" % thread_id)
    return

threads = []
for i in range(5):
    t = threading.Thread(target=run, args=(i,))
    threads.append(t)
    t.start()
    t.join()
```

当我们执行这段代码，输出如下:

```
function called by thread 0
function called by thread 1
function called by thread 2
function called by thread 3
function called by thread 4
```

然后来解释一下  `threads.append(t)` 、`t.start()` 和 `t.join()` 这三行代码: `threads.append(t)` 用来向追加线程的实例，在这之后我们调用 `t.start()` 之前加入的线程会开始调度运行，而 `t.join()` 会等待所有的线程执行完毕后退出。

### 使用子类的形式创建线程

除了使用函数的形式去定义线程之外，我们还可以使用继承 Thread 基类的方式，使用面向对象的形式来定义一个线程类:

```python
import threading
class MyThread(threading.Thread):
    def __init__(self, threadID, name):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
    def run(self):
        print(f"Starting, {self.name}")


t1 = MyThread(1, 'MyThreadFirst')
t2 = MyThread(2, 'MyThreadSecond')
t1.start()
t2.start()
t1.join()
t2.join()
```

这么做的核心有三个步骤:

1. 定义一个线程类，继承 `threading.Thread` 
2. 定义 `__init__` 方法实现自定义的类的实例参数
3. 定义 `run` 方法，作为线程执行的入口

### 对线程自定义命名

上文已经说到，使用 `threading.Thread` 函数中的 `name` 参数可以对创建的线程命名，如果没有指定这个参数就会按照默认的规则,即 Thread_N 的方式命名。下面的例子中演示了这个知识点:

```python
import threading
import time

def first_thread():
    print(f'Thread Name is {threading.current_thread().name}')
    time.sleep(2)
def second_thread():
    print(f'Thread Name is {threading.current_thread().name}')
    time.sleep(2)

t1 = threading.Thread(name='first_thread', group=None, target=first_thread)
t2 = threading.Thread(name='second_thread', group=None, target=second_thread)

t1.start()
t2.start()
t1.join()
t2.join()
```

上面的示例中，我们启动了两个线程。每个线程除了打印当前线程的名字外，还都延时了两秒。使用了 `threading.current_thread().name` 获取了它线程的名称。

### 参考资料

1. [《Python 并行编程手册》](https://book.douban.com/subject/30203042/)