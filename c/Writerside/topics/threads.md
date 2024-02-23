# 多线程

在编程中，并发编程一直都是一个难点。所以这篇文章希望为大家从事并发编程打下牢固的基础，通过 C 语言，把并发编程中一些基础内容讲解清楚，为更深入地学习并发编程做好准备。传统的 C 语言只支持进程层级的并发，直到 C11 标准才提供了线程级别的并发模型。

### 编译平台对多线程的支持 {id="platform"}

到目前为止, 主流的编译平台对 C11 中的多线程特性的支持并不是很好。MSVC 的新版本可能已经支持了 C11 标准和 C17 标准，但是不确定是否支持多线程。而在 GCC 编译器中，它的开发者认为多线程并不属于编译器的职责所以也并没有支持。

所以，如果要使用多线程。可以使用一个名为 TinyCThread 的库, 正如其名，它是对 C11 线程的轻量级的开源实现。可以工作在 Windows、Mac OS 以及 Linux 上。

### 使用 TinyCThread {id="tiny-c-thread"}

这个库是开源于 [GitHub](https://github.com/tinycthread/tinycthread), 可以通过链接来访问并下载代码，放在项目的工程目录中，加以使用。前文也提到了，它是一个非常轻量级的实现，这一点从代码文件上也可以体现出来，只有一个 C 文件和头文件。所以，也可以直接复制这两个文件到我们的工程目录中去。

如果要正常编译，需要修改 Cmake 配置，示例如下：
```
include_directories(tinycthread/include)
add_executable(untitled main.c tinycthread/tinycthread.c)
```
引入这个库如下:
```c
#include <tinycthread.h>
```

### Hello World {id="hello-world"}

然后我们来写一个 Hello World 的实例，通过创建一个线程来输出 “Hello World” 字符:
```c
#include <stdio.h>
#include <tinycthread.h>		// 引入头文件

int SayHello(char *name) {
    // thrd_current 函数返回的是当前线程的 ID，使用十六进制输出 %#x
    printf("Run in new thread[%#x]: %s\n", thrd_current(), name);
    return 0;
}
```
如何创建线程运行这个 `SayHello` 函数呢?如下:
```c
int main() {
    thrd_t new_thread;
    // thrd_create 函数用来创建一个线程，接受三个参数，
    // 分别是 thrd_t 线程的ID、线程执行的函数以及函数的参数，如果没有参数，可以传入 void
    int result = thrd_create(&new_thread, SayHello, "Hello World");
    // 返回的 result 是一个整形，可以使用常量 thrd_success 来判断是否创建成功
    if (result == thrd_success) {
        printf("Run in Main thread[%#x], created new_thread[%#x]\n",
               thrd_current(), new_thread);
    }
}
```
但是运行之后，这个程序的输出可能如下:
```c
Run in Main thread[0x150bce00], created new_thread[0x92d3000]
```
你是否感到奇怪，为什么没有输出 `SayHello` 函数中的内容？因为主程序比线程执行的快。我们创建的线程还没有等到它执行，主程序就已经结束了。怎么办呢？让主程序等一等嘛:
```c
// 让线程休眠一段时间，接受一个参数，传入匿名的 timespec 结构体，100000000 表示 100 毫秒
// 第三个参数我们传入了 NULL, sleep 并不是精确的计时，也就是说并不一定会等到 100 毫秒
// 如果需要获取精确的休眠时间，就可以通过传入变量来获取它
thrd_sleep(&(struct timespec) {.tv_sec = 0, .tv_nsec = 100000000}, NULL);
```
然后我们运行这个程序，并查看输出结果，类似如下输出，具体的地址和我是不一样的:
```
Run in Main thread[0x9931e00], created new_thread[0xd045000]
Run in new thread[0xd045000]: Hello World
```
> 注意: 如果是 Linux 环境，需要在编译参数中加入 `pthread` 。使用 Cmake 可以加入条件:

```c
if (UNIX OR Linux) 
    target_link_libraries(${name} pthread)
endif()
```
但是，在程序中使用 `thrd_sleep` 函数来等待线程结束并不是一个好的做法。因为很有可能线程的执行时间要比 sleep 的时间更长，那就尴尬了。所以，我们有更好的做法:
```c
int res;	// 定义变量接收线程函数执行完成之后的返回值
// 接收两个参数，来等待线程的结束，第一个是 thrd_t 类型的线程 ID
// 第二个是一个引用变量接收线程函数的返回值
thrd_join(new_thread, &res);
```
这样一来，就可以更加优雅地等待线程执行结束了。还有一种情况，我可能并不关心线程执行的结果，我们可以使用 `thrd_detach` 函数:
```c
thrd_detach(new_thread);
```
需要注意的是，如果线程函数的传参和返回值都比较复杂，不是一个基本的类型，可以使用结构体。

### 线程安全问题 {id="threads-safe"}

并发编程中，常常需要对一些共享资源进行访问，这会导致出现安全问题，并且难以排查。引起线程不安全的原因有很多，常见的原因如下。下文针对这些原因，分别举例说明:

- 对共享资源进行 **非原子** 的并发访问
- 不同线程之间的代码 **可见行 **问题
- 线程内部代码编译时的 **重排序 **问题

#### 对共享资源进行非原子的并发访问 {id="share-resource"}

我们来写一个计数的例子，来展示一个不安全的实例，然后再去分析它为什么不安全:
```c
// 这个 count 变量就是一个全局的资源，在整个 C 文件中都可以访问
// 如果代码是串行的，那么这不会有问题，反之并行就会导致不安全的访问
int count = 0;
```
写一个线程函数，对共享资源 `count` 变量进行递增:
```c
int Counter(void *arg) {
    printf("Counter...\n");
    // 循环 100 万次，对变量 count 递增
    for (int i = 0;i < 1000000; i++) {
        count++;
    }
    return 0;
}
```
然后创建两个线程，并行执行 `Counter` 函数，再去查看结果:
```c
int main() {
    thrd_t t_1, t_2;
    // 创建两个线程并行对 count 变量计数
    thrd_create(&t_1, Counter, NULL);
    thrd_create(&t_2, Counter, NULL);
    // 等待两个线程结束
    thrd_join(t_1, NULL);
    thrd_join(t_2, NULL);
    // 理论上 counter 应该是 200 万，实际上呢？不知道
    printf("count is %d\n", count);
}
```
运行这个程序，结果肯定不是我们预想的 200 万，而且是随机的小于 200 万的数字。这是为什么呢？根本原因就在于 `count++` 这句代码是非**原子性操作**的？也就是说 CPU 在执行这句代码的时候不是一次性执行完成的，起码会分成三个步骤执行:

1. 获取 `count` 变量的值
2. 对 `count` 变量递增
3. 将递增的结果重新为 `count` 变量赋值

下面这张时序表就重现了这个过程:

| 时序 | t_1 线程的执行               | t_2 线程的执行               |
|----|-------------------------|-------------------------|
| T1 | 获取 count 的值, 值为 0       | 获取 count 的值, 值为 0       |
| T2 | count 递增，值为 1           |                         |
| T3 |                         | count 递增值，值为 1          |
| T4 | 将递增的结果重新赋值，此时 count 为 1 |                         |
| T5 |                         | 将递增的结果重新赋值，此时，count 为 1 |
| TN | ......                  | ......                  |

因为  `count++` 这个操作不是原子性的，所以在并行写入的时候，多个线程之间可能会覆盖彼此的递增结果，导致了最终的结果是小于预期的结果的。

#### 不同线程之间的代码可见行问题 {id="visible"}

共享资源的可见行指的是线程 A 对资源进行了修改，但是线程 B 不会立即看到，只有在 CPU 的缓存中的数据同步到内存之后才能看到。由此造成了线程对数据的错误访问，导致了最终结果的错误。

我们来写一个例子, 试图重现这个可见性的问题:
```c
int flag = 0;		// 立下 flag， 如果为真，则将 a 的乘积赋值给  x
int a = 0;			// 共享变量 a
int x = 0;			// 共享变量 x
// 在线程 T1 中对 a 和 flag 赋值，使得 flag 为真，a 为 2
int T1(void *arg) {
    a = 2;
    flag = 1;
    return 0;
}
// 在线程 T2 中对 x 赋值
int T2(void *arg) {
    while(!flag) {}	// 如果 flag 为假，则不断循环，直到真为止(说明 T1 已经执行)
    // 将 a 的乘积赋值给 x，理论上 T1 已经执行，所以 a 为 2，x 为 4
    // 理论结果，可能为 4 也可能为 0， 因为 a 可能存在可见行问题，导致 T2 读取不到它更新后的值
    x = a * a;	
    return 0;
}
```
我们来测试这两个线程的执行结果:
```c
int main() {
    thrd_t t_1, t_2;
    // 创建两个线程并行对 count 变量计数
    thrd_create(&t_1, T1, NULL);
    thrd_create(&t_2, T2, NULL);
    // 等待两个线程结束
    thrd_join(t_1, NULL);
    thrd_join(t_2, NULL);
    // 理论上 counter 应该是 200 万，实际上呢？不知道
    printf("count is %d\n", x);
}
```
我的测试结果一直都是 4，并没有出现过 0 的情况。所以我没能重现这个可见性的问题，但是我们不能排除会出现 0 的可能，这和编译器的实现有关。

#### 线程内部代码编译时的重排序问题 {id="resorted"}

不同的编译器编译之后，产生的汇编代码是不同的，编译器会对我们的代码进行优化，比如说对代码进行重排序。具体的优化我们可以通过 Compiler Explorer 这个工具来查看。我们改写 `T1` 线程函数，加入一句赋值:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/U6YHn6TisazFaTz0jsBM.png)

我们可以看到经过编译器的优化 `a  = 2` 这行代码在编译后的汇编中已经不存在了，看汇编指令的第 5 行，直接将 `a` 赋值成了 5，足见编译器非常聪明，通过对代码的重排序，减少指令的执行。

如果我们在编译选项中加入 `-O3`(字符O，非数字0），然后我们再来看看之前的代码中的循环的汇编指令:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/DxLqf74q5rxvnQjqsddP.png)

测试 `flag` 是否为真，如果为真则跳到 `.L4` 位置，否则则跳到了 `.L5` 位置，然后不断跳转到 `.L5` 这就是一个死循环。那么这个程序的结果可能就是死循环。但是我在 MacOS 中并依然没有重现这个现象。

> 注意: 在 Cmake 中加入编译优化参数: set(CMAKE_C_FLAGS "-O3")。


### volatile {id="volatile"}

`volatile` 这个关键字在 C90 标准中已经存在了，但是那时候还不支持多线程，所以它的立意和多线程无关。但是，它“很巧合”的可以被用来解决线程安全中的代码重排序问题。所以，我们来看看这个关键字。

在英语中，“volatile”这个单词含有“易变”的意思。而在 C 语言中，使用 `volatile` 关键字修饰的变量，编译器不会对其的读写顺序进行重排序。

比如上一章节中的示例程序中的变量我们使用它去修饰，然后再观察编译后的汇编指令，不会出现重排序的情况:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/zIcfSxZJ82j9EDPRRnTj.png)

通过对上面第 2 行和第 5 行汇编指令的解读，我们发现编译器会忠于我们的代码，不会对其进行重新排序优化。我们也需要对 `flag` 变量加上修饰，这样可以解决死循环的问题:
```c
volatile int flag = 0;		// 可以解决汇编指令中出现死循环的问题
```
但是需要注意的是， **`volatile` 并不能解决 CPU 缓存导致的线程间的数据可见行问题，它也不会保证访问的原子性，它在 C 标准中的用意是禁止编译器优化数据的读写操作。在其他的语言中，比如 Java 中， `volatile` 关键字的含义和 C 语言中是不一样的，是具备可见行的语义的。注意: 在 MSVC 编译环境下，赋予了 `volatile` 关键字强制刷新缓存的语义，可以保证其可见性。**
**
### 原子类型和原子操作 {id="atom"}

在上文中，我们举了一个计数器的例子来说明对共享资源进行非原子的并发访问，会导致线程不安全。其根本原因就是如下代码并不是原子性的:
```c
int count = 0;
count++;		// 不是原子性的
```
在 C11 标准中，已经提供了原子性的变量类型。所以只要修改变量类型，就可以得到正确的答案，如下示例:
```c
#include <stdatomic.h>

atomic_int count = 0;
count++;		// 这个操作就变成了原子性操作
```
其他的类型也都一样，比如 `atomic_flag` 之类的。然后我们来写一个在主线程中定时取消子线程取消的例子，先定义一个原子类型:
```c
 #include <stdatomic.h>
// ATOMIC_FLAG_INIT 是对 resume_flag 这个原子性的布尔值进行初始化，初始化值为 0
atomic_flag resume_flag = ATOMIC_FLAG_INIT;
```
接着来写一个线程函数，根据 `resume_flag` 标志来确定是否继续输出:
```c
int PrintNumbers(void *arg) {
    int current = 0;
    // 测试 resume_flag 是否为真，如果是则不断输出 current 递增后的值，比如  1, 2, 3......
    while(atomic_flag_test_and_set(&resume_flag)) {
        current++;
        printf("current is %d\n", current);
        // 每次休眠一秒
        thrd_sleep(&(struct timespec){.tv_sec=1}, NULL);
    }
    return current;
}
```
测试这个线程函数:
```c
int main(void) {
    atomic_flag_test_and_set(&resume_flag); // 设置标志为为 true
    thrd_t t;
    thrd_create(&t, PrintNumbers, NULL);        // 创建线程
    // 休息 5 秒
    thrd_sleep(&(struct timespec) {.tv_sec=5}, NULL);
    // 清除标志位，之后 atomic_flag_test_and_set 函数就会返回 false
    int last_number;
    atomic_flag_clear(&resume_flag);
    // 获获取线程函数的返回值
    atomic_flag_clear(&resume_flag);        // 再次恢复到 0
    thrd_join(t, &last_number);
    return 0;
}
```
这些原子性的指令都是通过 CPU 原子性指令来实现的，所以性能非常好，但有些也是使用加锁的方式实现，性能就会差一些。标志位非常适合做这一类的事情，但是不灵活不能随便读写。但是它可以通过 CPU 原子性指令来保证共享资源在不同的i线程汇总的原子性和可见性。

### 使用锁解决并发问题 {id="lock"}

前文也提到了，原子性的类型和操作大部分都是通过 CPU 的原子指令来完成的，所以性能比较好，还有一部分可能是通过锁来完成的，性能就差很多。所以说，锁是没有办法的办法。但还是经常要用，所以来说说吧。

在 C11 标准中，提供了一个锁的类型 `mtx_t` , 标准中并没有定义它具体的实现，它可以是一个整数。围绕着它展开的，有一系列的操作函数，如下表所示:

| 函数            | 作用              |
|---------------|-----------------|
| mtx_init      | 初始化锁            |
| mtx_destroy   | 销毁锁             |
| mtx_lock      | 加锁，会产生阻塞        |
| mtx_unlock    | 解锁              |
| mtx_timedlock | 加锁，但是可以设定超时时间   |
| mtx_trylock   | 尝试加锁，如果锁被占用，则结束 |

需要额外说明的是， `mtx_timedlock`  函数需要传入一个锁的类型，具体有三种类型，分别是:

- `mtx_plain` 类型：这就是最普通的锁，不可重入锁
- `mtx_timed` 类型：支持设定超时时间，如果到时候仍然没有获取锁，就结束
- `mtx_recursive` 类型：可重入锁，当需要递归调用函数重复加锁的时候，就使用这个类型

说了那么多，我们来举一个例子，依旧是上文说到的计数器的例子。上文中，我们已经采用了原子类型来保证了线程的安全，并且这是一种高性能的做法。

首先我们需要声明一个锁:
```c
mtx_t mutex;
int count = 0;
```
然后重构计数器的线程函数, 在非原子操作 `count++` 的前后加解锁，使之成为原子性操作:
```c
int Counter(void *arg) {
    for (int i = 0; i < 1000000; i++) {
        mtx_lock(&mutex);            // 加锁
        count++;
        mtx_unlock(&mutex);      // 解锁
    }
    return 0;
}
```
最后我们来测试这个并发计数，创建两个线程并发执行。与之前的版本相比，只是多了初始化锁和销毁锁的操作:
```c
int main(void) {
    thrd_t t_1, t_2;
    mtx_init(&mutex, mtx_plain);     // 初始化锁
    thrd_create(&t_1, &Counter, NULL);
    thrd_create(&t_2, &Counter, NULL);
    thrd_join(t_1, NULL);
    thrd_join(t_2, NULL);
    mtx_destroy(&mutex);     // 销毁锁
    printf("count is %d\n", count);		// 没有疑问，输出 count 为 200 万
}
```

### Thread Local {id="thread-local"}

**Thread Local（线程存储期），**当一个变量使用 `_Thread_local` 类型关键字修饰的时候，就可以获得线程存储期。这一类变量**在线程中，会如同自动变量一样，在线程一开始被创建、线程结束前被销毁，线程与线程之间互不影响。**
```c
_Thread_local int count = 0;		// 被声明为拥有线程存储期的变量
int Counter(void *arg) {
    for (int i = 0; i < 1000000; i++) {
        count += 1;
    }
    printf("%d\n", count);		// 每个线程都会输出 1000000，互不干涉
    return 0;
}		// 省略对这个线程函数的对应的线程的创建代码
```
> 注意：count 这个变量，对于主线程而言也会存在一个副本，和其他线程无关。


**通过线程安全期这种做法，将一个全局的变量转为线程私有的变量，从根本上解决了线程安全的问题，即不存在共享资源。 **
**
然后我们来使用另外一种更为灵活的方式来创建拥有线程存储期的变量，看上去**这种方式并不需要在全局作用域内创建变量，而是在线程函数中创建并且通过 Key 在线程之间绑定。**为此，提供了下面这些函数:

| 函数         | 作用                     |
|------------|------------------------|
| tss_set    | 使用它来绑定 key 和变量         |
| tss_get    | 使用它来获取对应的 key 绑定的变量的值  |
| tss_create | 使用它来创建一个将用于变量绑定的 key   |
| tss_delete | 使用它来销毁已经不被使用的变量绑定的 key |

还是通过计数器的例子来说明这些函数的使用。首先我们需要一个 key:
```c
tss_t count_key;
```
接着创建计数器函数:
```c
int Counter(void *arg) {
    int *count = malloc(sizeof(int*));
    *count = 0;
    // 使用 tss_set 函数来绑定 key 和变量，通过 thrd_success 来判断是否成功
    if (tss_set(count_key, count) == thrd_success) {
        for (int i = 0; i < 1000000; i++) {
            *count += 1;
        }
        // 使用 tss_get 获取值，因为是 void * 类型的，所以强转为 int *, 因为是指针所以要解引用
        printf("%d\n", *((int *)tss_get(count_key)));    // 两个线程都输出 1000000, 互不干扰
    }
    return 0;
}
```
最后在主函数中测试，并创建 key, 结束之后再销毁：
```c
int main(void) {
    // 创建 key, MyFree 函数用于线程结束时候销毁对应变量资源
    if (tss_create(&count_key, MyFree) == thrd_success) {
        thrd_t t_1, t_2;
        thrd_create(&t_1, &Counter, NULL);
        thrd_create(&t_2, &Counter, NULL);
        thrd_join(t_1, NULL);
        thrd_join(t_2, NULL);
        // 如果在线程推出之前(join之前)主动销毁，则不会调用 MyFree, 在调用前需要确保资源不再使用
        tss_delete(count_key);
    }
}
```
使用 `tss_create` 函数去创建一个 key 的时候，需要制定一个回调函数:
```c
void MyFree(void *ptr) {
    free(ptr);		// 释放对应的指针
}
```

### 线程的返回值 {id="return-value"}

当我们定义一个线程函数的时候，可以指定返回值。当我们使用 `thrd_create` 函数创建一个线程的时候，传入的线程函数是一个 `thrd_start_t` 的类型，其原型如下:
```c
typedef int (*thrd_start_t)(void *arg);
```
是不是感觉很奇怪？难道我只能够返回整形嘛？不是的，返回值确实只能是整形，但是我们可以借助参数指针来实现目的,因为它的参数是 `void *arg` ,任何类型都可以。首先，定义一个复杂的结构体:
```c
typedef struct {
    char *result;		// 把传参和结果都可以放在这个结构体中
} ComplexArg;
```
定义一个线程函数, 简单的给结果赋值，然后返回:
```c
int func(ComplexArg *arg) {
    arg->result= "Hello World";		// 结构体中可以是任何的返回值
    return arg;
}
```
然后，在主函数中创建线程并使用结果:
```c
thrd_t thrd_1;
ComplexArg complex_arg;		// 创建一个结构体类型的参数，当中包含结果
thrd_create(&thrd_1, &func, &complex_arg);
int result;         // 此处的 result 只表示状态，而非结果
thrd_join(thrd_1, &result);
printf("%s\n", complex_arg.result);		// 输出结果
```

### 案例: 下载文件 {id="case"}

这里提供了一个下载文件的案例，当中并没有实现下载文件，但是将上文中提到的所有的知识点都串联了起来。这里把代码贴出来，代码中的注释已经很详细了:
```c
#include <stdio.h>
#include <tinycthread.h>

// 定义文件下载的上下文结构体
typedef struct Context {
    int download_left;      // 剩余的下载文件的数量
    mtx_t mutex;     // 定义锁，用来保障 download_left 变量的安全访问
} Context;

// 定义下载请求的结构体
typedef struct DownloadRequest {
    struct Context *content;        // 文件下载的上下文结构体
    char const *url;                // 下载的资源路径
    char const *filename;           // 下载的文件名称
    int progress;                   // 下载的进度
    int interval;                   // 剩余的时间
    void (*callback)(struct DownloadRequest *);      // 结果回调函数，用于与主线程通信
} DownloadRequest;

// 定义该函数用来模拟下载过程中的等待耗
void wait(long milliseconds) {
    long seconds = milliseconds / 1000;
    long nanoseconds = (milliseconds % 1000) * 1000000L;
    thrd_sleep(&(struct timespec) {.tv_sec=seconds, .tv_nsec=nanoseconds}, NULL);
}

// 定义下载文件的线程函数，接受一个下载结果的结构体作为传参
int DownloadFile(struct DownloadRequest *request) {
    // \r 的作用是将光标移动到行首，并清空光标到行首之间的内容，以此来实现在一行内不断刷新进度的效果
    printf("\rDownloading file from: %s into %s ...", request->url, request->filename);
    // 模拟下载进度
    for (int i = 0; i < 100; i++) {
        request->progress = i;
        wait(request->interval);
    }
    // 下载完成之后，执行回调函数，并传入请求信息
    request->callback(request);
    return 0;
}

// 定义回调函数
void DownloadCallback(struct DownloadRequest *request) {
    mtx_lock(&request->content->mutex);     // 对资源的访问加锁
    request->content->download_left--;      // 因为有锁，所以这是原子操作
    printf("\rDownload file from: %s into %s successfully, left: %d",
           request->url,
           request->filename,
           request->content->download_left);     // 打印下载完成的信息
    mtx_unlock(&request->content->mutex);   // 访问结束之后解锁
}

#define  DOWNLOAD_TASKS 5      // 定义宏，一共有多少个下载任务

int main(void) {
    char *urls[] = {        // 模拟假数据，文件的下载地址
            "https://dev-test.cn/file1",
            "https://dev-test.cn/file2",
    };
    char *filenames[] = {       // 模拟假数据，文件的名称
            "downloads/file1",
            "downloads/file2",
    };
    DownloadRequest requests[DOWNLOAD_TASKS];     // 请求的数组
    struct Context context = {.download_left = DOWNLOAD_TASKS};
    mtx_init(&context.mutex, mtx_plain);
    for (int i = 0; i < DOWNLOAD_TASKS; i++) {
        requests[i] = (struct DownloadRequest) {
            .content = &context,
            .url = urls[i],
            .filename = filenames[i],
            .progress = 0,
            .interval = i * 50 + 100,
            .callback = DownloadCallback
        };
        thrd_t t;
        thrd_create(&t, DownloadFile, &requests[i]);
        thrd_detach(t);     // 不要阻塞等待
    }

    while(1) {
        // 将全局资源赋值给一个本地变量，之后使用本地变量就不会有线程安全的问题了，可以减少锁的等待时间
        mtx_lock(&context.mutex);       // 读取加锁
        int download_left = context.download_left;
        mtx_unlock(&context.mutex);     //读取完毕解锁
        // 判断是否已经全部下载完成
        if (download_left == 0) {
            break;
        }
        printf("\r");
        // 打印进度，这里就是上文中的通过线程函数的传参来传递信息的例子
        for (int i = 0; i < DOWNLOAD_TASKS; i++) {
            printf("%s -- %3d%% \t", requests[i].filename, requests[i].progress);
        }
        fflush((stdout));       // 刷新缓冲区
        wait(500);      // 五百毫秒刷新一次
    }
    mtx_destroy(&context.mutex);

    return 0;
}
```