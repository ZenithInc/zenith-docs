# 函数是一等公民

Python 是一种支持多范式的编程语言，比如面向对象、面向协议、函数式编程等。这一篇文档主要描述在 Python 中，有关于函数式编程的特性。

### 为什么说函数是一等公民

说到函数式编程，我们最常听到的一句话就是 **“函数是一等公民”** 。那么在编程语言中，什么是一等公民呢？

这就要从“公民”这个词说起。公民这个词有两个主要的内涵，一是权力，二是义务。作为一个公民，在这个国度我们有很多的义务，比如说义务服兵役、纳税是公民义务。在履行义务的同时，我们也享有一定的权力，比如说的生命安全不受侵犯、我们的私有财产权不受侵犯.....那么，这就是我理解的所谓共鸣。

那么什么是一等公民呢？说的是地位。比如说，在电影《精武英雄》中，经典的一句台词是“中国人与狗不得入内”。那么在这句侮辱性的台词中，中国人就不是一等公民。具体的表现就是不享有和洋人一样的自由的权力。所以，所谓的一等公民在社会中指的就是人人平等。

言归正传，在编程中什么是一等公民呢？首先我们要说公民有哪些组成。在传统的语言中，各种类型的值、对象都是公民，他们有什么样的权力呢？可以在函数或方法之间传递、可以赋值给一个变量、可以在函数或方法中作为返回值等等。

说到这里，大概也就明白了为什么说函数也可以作为一等公民。意思就是说：**函数也可以像整型、字符串型、对象类型作为一种类型，可以有字面量、给变量赋值、可以在函数或方法间传递、可以在函数或方法中作为返回值.....**

### 函数是一等公民的体现

既然说函数是一等公民，那么我们就用代码来看看，它是如行使其权力的。

#### 函数可以赋值

是的，既然是一等公民，那么赋值就是一件理所当然的事情啦:

```python
def greet(name):
    print(f'Hello, {name}')
func = greet
func('bob') # 输出: Hello, bob
```

#### 函数可以存储与其他的数据结构中

然后，函数也可以存储在其他的数据结构中，比如字典, 下面用一个简单的计算器为例子:

```python
def add(num1, num2):
    return num1 + num2
def reduce(num1, num2):
    return num1 - num2
calc = dict(add=add, reduce=reduce)  # 将函数存入字典
num1 = 2 # 定义算数
num2 = 1
operate = 'add' # 定义操作符
print(calc[operate](num1, num2))  # 通过操作符找到合适的算法函数
operate = 'reduce'
print(calc[operate](num1, num2))
```

上面的示例代码中，我们定义了两个计算器的方法，分别是加法和减法，然后我们将这两个方法存储在字典这个数据结构中，通过改变 operate 操作符来灵活的调用 calc 中的预置方法实现计算。

#### 函数可以在函数之间传递

另外我们还可以让函数在函数之间传递，就是其他的变量类型一样自然。仍然是上面的例子，我们允许自定义计算方法:

```python
def calc(num1, num2, func):
    return func(num1, num2)
def add(num1, num2):
    return num1 + num2
print(calc(1, 2, add))
```

上面的示例代码中，我们定义了一个 calc 的函数，但是在这个函数中并没有明确采用什么样的算法，函数的调用者可以通过 func 的传参自定义。而后，我们又定义了一个名为 add 的方法，可以通过 calc 的传参传入，并由其进行调用完成计算。

**在 Python 中，能够接受函数为传参的函数，我们称之为高阶函数。 **在 Python 中，map 就是这样一个高阶函数。下面这个例子演示了使用 map 求平方:

```python
list_ = [1, 2, 3, 4, 5, 6]
def square(value):
    return value * value
print(list(map(square, list_)))
```

#### 函数可以嵌套

我们可以在函数中定义函数，这样的函数也被称之为 **内部函数**，或者叫做 **嵌套函数**。如下示例:

```python
def print_log(message):
    def print_(text):
        print(text)
    print_(message)
print_log(message='success')
```

这个函数的定义本身并没有太大的意义，只是为了说明函数是可以嵌套的。上面的示例演示了可以定义一个内部函数，然后在内部调用。我们也可以定义一个内部函数，并将这个内部函数作为返回值:

```python
def print_log():
    def print_(message):
        print(message)
    return print_
print_log()('success')
```

#### 闭包

在上面的基础上，我们来写一个计数器:

```python
def counter():
    ## 定义了一个 counter 的内部变量
    number = [0]
    def print_():
        ## 对这个变量进行累加
        number[0] = number[0]+1
        return number[0]
    return print_
## 不断调用这个返回的函数
func = counter()
for _ in range(5):
    print(func())   ## 输出 1 到 5
```

上面这种形式称之为闭包，所谓闭包就是延续了父函数的状态。按照道理来说 `number`是一个局部变量，当 `counter`函数执行完成之后，也就是 `return number[0]`了之后，这个 `number`的生命就走到了尽头。

但实际上并没有，这个 `number`被 `print_`这个内部函数带到了外部， `number`的生命也随之延续。这就是闭包，闭包可以复制定义它的父函数的状态(也就是变量)。

#### 对象也可以作为函数调用

我们说所有的函数都是对象，但不能说所有的对象都是函数。这句话的潜台词就是一部分的对象是函数(准确的说像函数一样被调用)，只要在对象内部实现了 `__call__`方法即可。

现在窗外面下着大雨，我就用写一个雨的对象，打印雨量的大小:

```python
class rain:
    def __init__(self, rainfall):
        self.rainfall = rainfall

    def __call__(self):
        print(self.rainfall)

rain('heavy')()  ## 输出: heavy
```

正常输出了 `heavy`，是的窗外面的雨很大，一连下了好几天。

