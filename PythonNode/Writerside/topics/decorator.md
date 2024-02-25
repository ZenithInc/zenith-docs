# 设计模式之装饰器

从一个小需求说起: 我需要实现一个函数，函数的功能非常简单，就是打印 Hello World。我很快就可以实现这个需求:

```python
def say_hello():
    print('Hello World')
```

然后，新增了一个需求，打印 Hello, Python。这也不是问题:

```python
def say_python():
    print('Hello Python')
```

最后，老板要求在打印 `Hello World` 或者 `Hello Python` 之前，首先需要打印 `Hello Boss` 。这时候，这个需求就会有很多中实现方法。下面我们来试试看吧。

### 第一种方法: 修改两个函数

第一种方法，也是最容易想到，最容易实现的方法，就是修改两个函数:

```python
# 第一个方法
def say_hello():
    print('Hello Boss')
    print('Hello World')
# 第二个方法
def say_python():
    print('Hello Boss')
    print('Hello Python')
```

这么写从功能上来说并没有什么问题，但是违反了面向对象编程的开闭原则， **对修改封闭，对扩展开放** 。如果我们下次需求变成了 `Hello Boss'wife` 呢？还要修改这两个地方，如果类似的地方有几十处呢？改到要吐。

### 第二种方法: 从新写一个方法

第二种方法，我们是为了解决上一种方法中的缺陷，即如果要修改则要改很多地方。

```python
# 新增一个方法
def greet_boss():
    print('Hello Boss')
# 另外两个方法不变
def say_hello():
    print('Hello World')
def say_python():
    print('Hello Python')
# 先行调用新增的方法
greet_boss()   # Hello Boss
say_hello()    # Hello World
greet_boss()   # Hello Boss
say_python()   # Hello Python
```

这样写好像好一些了，然后呢？但是这种写法还可以优化一下，延申出第三种做法。

### 第三种方法: 传递函数

Python 是一门多范式的语言，支持面向对象的特性，也支持函数式编程的特性。我们在 Python 中可以试函数为一等公民，四处传递。

```python
def greet_boss(func):
    print('Hello Boss')
    func()
# 其他两个方法不变
# 调用方法改变
greet_boss(say_hello)
greet_boss(say_python)
```

相比前两种，看上去要好一些了。还有没有其他方法呢？装饰器模式。

### 第四种方法：装饰器

装饰器是一种面向对象编程中的模式，也可以理解成是套路。模式是由固定的应用场景的，就像常见的套路一样，比如有专门哄骗小姑娘的套路。

首先定义一个函数:

```python
def decorator(func):
    def wrapper():
        print('Hello Boss')
        func()
    return wrapper
```

上面这个函数就叫做装饰器的函数，是不是非常像第三种写法呢？只不过我们在函数内部又封装了一个函数，然后把函数返回了。接下来就来使用这个装饰器:

```python
decorator(say_hello)()
decorator(say_python)()
```

因为返回的是函数嘛，所以在后面直接加上 `()` 就表示调用了。

第三种方法和第四种方法存在一个问题，就是 decorator 方法并没有和 say_hello 以及 say_python 关联起来。

### 第五种方法: 使用装饰器的语法糖

经过上面这几种方式，我们发现原本一件简简单单的事情，逐渐变得复杂起来了。但是在实际的编程中，更为复杂的东西叫做现实。**因为编程是为了模拟现实，现实越多变、越复杂，编程就越需要灵活、需要简化。**

因为上面这些三四两种方法都太过于繁琐，所以 Python 提供了装饰器的语法糖，让我们使用装饰器可以更加的灵活、简便:

```python
def decorator(func):
    def wrapper():
        print("Hello Boss")
        func()
    return wrapper

@decorator
def say_hello():
    print('Hello World')

@decorator
def say_python():
    print('Hello Python')

say_hello()
say_python()
```

虽然我们定义 decorator 方法亦然很复杂，然是我们调用变得简单了。只需要在原有方法的申明前面加上 `@<装饰器方法的名称>` 就可以直接调用原有的方法，并且实现对原有方法的扩展(即加上了 Hello Boss 的输出)。 **所以，装饰器的最大的意义在于调用，而不是定义。**

定义复杂没有关系，只写一次。而调用复杂是不能忍受的，因为到处都要调用。**所以，评价接口、方法定义的好坏，最简单最直观的方法就是看调用是否方便。**
**

### 为装饰器增加传参

被装饰器修饰的方法，如果要添加参数怎么办？

```python
def decorator(func):
    def wrapper(*args, **kw):
        print("Hello Boss")
        func(*args, **kw)
    return wrapper
```

为 wrapper 函数加上可变参数 `*args` ,这样就可以传递任意的参数给被装饰的函数啦。

### 保留被装饰的函数的元数据

我们来运行下面这段示例代码，定义了一个名为 `print_log` 的装饰器，它会在函数执行之前打印函数的名称。然后我们定义了一个被 `print_log` 修饰的函数，名为 `greet` , 它会执行打印“execc”字符。

```python
def print_log(func):
    def wrapper():
        print(f'start...{func.__name__}')
        return func
    return wrapper

@print_log
def greet():
    print('exec')

print(greet.__name__) ## 输出: wrapper
```

执行这段代码之后，我们会发现第 11 行输出的竟然是 “wrapper”，而不是 “greet”。 **是的，被装饰器修饰之后，函数的元数据就会被装饰器函数覆写** 。所以说，美其名曰是装饰，不如说是替换了原有的函数去执行。

这带来的问题就是，被装饰的函数并不容易去调试。要解决这个问题，需要使用 Python 的基本库中的 `@functools.wraps` 这个装饰器，例如:

```python
import functools
def print_log(func):
    @functools.wraps(func)
    def wrapper():
        print(f'start...{func.__name__}')
        return func
    return wrapper
```

之后我们调用 `print(greet.__name__)` 或者 `print(greet.__doc__)` 打印出来就回是 `greet` 函数本身的元数据了。