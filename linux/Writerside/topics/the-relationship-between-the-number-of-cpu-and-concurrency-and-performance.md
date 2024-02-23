# CPU核数、并发数以及性能的关系

这篇文档描述了 CPU 核心数量、并发数量以及性能的关系。主要有统筹方法以及阿尔达姆定律。

## 统筹方法 {id="overall-approach"}

我国著名的数学家华罗庚使用泡茶的例子来说明统筹法，如下图所示:

![image](http://file-linker.oss-cn-hangzhou.aliyuncs.com/DZizEvo5Z9LQu2gIZVn5.jpg)

很明显使用统筹法，有相互依赖关系的部分使用串行，而没有相互依赖关系的部分使用并行，这样统筹安排节省了时间，提升了效率。使用统筹法对我们的程序并行计算是有很好的指导作用的。

这个定律是计算机界的一个经验法则，它代表了处理器并行运算后效率的提升能力。理论最大加速比是通过下面这个公式计算而来的:

<code-block lang="tex">
\begin{equation}
s(n) = \frac{1}{(1 - p)  + \frac{p}{n}}
\end{equation}
</code-block>

这个公式中各个元素的含义如下:

* N : 处理核心数的数量
* P：可以加速时间占比

我们来写一个程序来举例:
```Python
import time
from concurrent.futures import ProcessPoolExecutor

def _task():
    time.sleep(1)

if __name__ == '__main__':
    start = time.time()
    for _ in range(10):
        _task()
    end = time.time()
    print('took: {}'.format(end - start))
    pool = ProcessPoolExecutor(10)
    start = time.time()
    futures = []
    for _ in range(10):
        future = pool.submit(_task)
        futures.append(future)
    for future in futures:
        future.result()
    end = time.time()
    print('took: {}'.format(end - start))
```
运行程序之后，我们得到了使用并行优化前后的时间占比，P 的计算如下:
```text
P = 1.1267099380493164 /  10.02167558670044  ~= 0.1124
```

然后我们将 P 代入到公式中去，就得到了最大优化占比，如下:
```text
S(10)  =  1 / ((1 - 0.112427) + 0.112427 / 10) ~= 1.1125
```

我们看到最大优化占比和我们的实际优化占比是非常接近的。如果我们不断的加大这个公式中的 N 的数值，就会得到一个曲线，发现 N 的数量并不是越大越好的。如下图所示:

![image](http://file-linker.oss-cn-hangzhou.aliyuncs.com/n8KOMAcOaNQQQefe0iH3.png)

## 总结 {id="summary"}

通过上面的分析，我们知道并不是线程数量越多，效率就越高的。线程的数量和 CPU 核心数量是有关系的，一般我们会采用下面的经验值:

* CPU 密集性的程序，我们采用 N + 1
* IO 密集性的程序，我们采用 2N + 1

线程过多，会引起线程之间去竞争 CPU 资源，会抵消一部分的优化效率。另外，线程过多会有上下文频繁的切换，这也是需要我们去考虑的成本。