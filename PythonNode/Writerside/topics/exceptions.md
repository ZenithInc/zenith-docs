# 异常处理

我们总是希望生活是美好的，总是希望明天充满了希望，但现实生活也总是希望和失望相互交a织。在程序中也是如此，我们希望程序可以从头到尾没有任何问题，但我们这么天真的想法本身就是问题。

## 什么是异常 {id="what"}

程序是现实世界的映射，现实世界中我们的生活也总是充满未知的异常。在程序中异常是不可避免，比如程序需要读取某个文件，但是不知道为什么这个文件不见了，可能是被谁删了。再比如程序预期用户输入一个数字，结果输入了一个字符串。

早期，异常的处理很简单，类似于 C 这样的语言通过整型的返回值来处理异常。如果返回 0 则表示正确、返回 1 则表示文件不存在、返回 2 则表示用户输入错误，以此类推。

但是这种处理方式存在两个问题：

* 每一个调用这个方法的地方都需要进行异常错误码的判断，如果弄错了错误码、或者没有检查错误码，一旦发生错误就难以排查。

* 如果 A 程序调用了 B 程序，B 程序调用了 C 程序，那么 A 和 B 可能都需要处理 C 返回的错误码。如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/b2gHm4jnjZeiiH3RPBoV.jpeg" alt="two problems" />

而异常机制就是为了解决上面这个问题的，如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/vvfOgk9uJod0aevvz4wQ.jpeg" alt="exception" />

## 抛出异常 {id="throws"}

于是，在生活中，我们开始学着为自己找好退路。而在编程中，我们也会尽可能去假设会出现问题，提前**捕获异常(Catch exception)**。

最简单的一个例子，除数不能为 0，但却写出了这样的代码:

```shell
$ python -c "print(1/0)"
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ZeroDivisionError: division by zero
```

然后程序给你一个无情的耳光，没办法再执行下去了。有的同学会说，我知道除数不能为 0，所以我不会写出这样的代码的。但是，程序是由万千的变量和分支组成的，很多数据也是来源于我们的用户或者第三方是我们不能掌控的。 **所以，异常是不可能避免的，但是可以提前预防的** 。

再比如说，当使用 Python3 去运行 Python2 的某些语法的时候:

```shell
$ python2 -c "print 'Hello World'"
Hello World
$ python3 -c "print 'Hello World'"
  File "<string>", line 1
    print 'Hello World'
                      ^
SyntaxError: Missing parentheses in call to 'print'. Did you mean print('Hello World')?
```

Python3 是一个不能向下兼容的版本，所以比如 `print 'Hello World'` 这样的语句在 Python3 中是无法执行的。抛出了一个异常 —— SyntaxError 。

## 异常捕获 {id="exception_catch"}

当我们预感代码可能会抛出异常的时候，可以捕获异常进行处理，记录日志:

```python
try:
    fdajljfdjalfjdlkajfdasjfldsa
except:
    print('syntax error')
print('continue')
```

另外，我们还可以对特定的异常进行捕获:

```python
try:
    1 / 0
except (ZeroDivisionError):
    print('Zero Division Error')
except:
    print('syntax error')
print('continue')
```

通常，发生异常的时候，我们会希望能够提供更多的异常相关的信息, 我们可以使用 `as` 关键字来获取 **异常对象（没错，异常也是对象) :**

```python
try:
    1 / 0
except ZeroDivisionError as e:
    print('Zero Division Error:' + e.__class__.__name__) # Zero Division Error:ZeroDivisionError
except:
    print('syntax error')
print('continue')
```

另外，Python 还提供了两个关键字, 分别是 `else` 和 `finally` ：

```python
try:
    1 / 0
except ZeroDivisionError as e:
    print('Zero Division Error:' + e.__class__.__name__)
except:
    print('syntax error')
else:
    print('normal')
finally:
    print('exception')
```

`else` 只有在没有发生异常的时候会执行，而 `finally` 是不管有没有发生异常，都会执行。

上面看到的示例都是 Python 解释器抛出的异常，我们自己也可以抛出异常, 使用 `raise` 关键字:

```shell
python3 -c "raise TypeError('Type Error')"
```

上面的 TypeError 也是 Python 自带的，我们可以自定义异常类，继承 BaseException 类。

```python
class UserNotFound(Exception):
    def __init__(self, user_id):
        super().__init__("User not found ${}".format(user_id))
        self.user_id = user_id

raise UserNotFound(10010)
```

执行这段代码，抛出自定义的异常如下:

```shell
$ python3 test.py
Traceback (most recent call last):
  File "test.py", line 6, in <module>
    raise UserNotFound(10010)
__main__.UserNotFound: User not found 10010
```

## 深究 finally

在 Python 中，为了避免资源在异常情况下没能释放，所以有了在 try 语句块的后面加上 finally 的做法。不管代码是否发生异常，都会执行 finally 的语句块。

但是如果我们继续深究 finally ，会有一些细节可能被我们忽略了。所以，本文希望可以挖掘出这些细节，避免因为对他们的忽略导致一些难以排查的问题的出现。

我们用如下的示例代码来解答一些问题:
```Python
def test():
    try:
        print("a")
        return 0
    finally:
        print("b")
        return 1

print(test())
```

根据这段代码以及运行的结果，我们来回答如下问题:

1. 可以在 `finally` 语句块中加入 `return` 吗？可以的，运行这段代码不会出现任何错误。
2. 第 4 行的 `return` 和第 7 行的 return 会执行哪个? 执行第 7 行的，所以第 9 行打印的结果是 1。

上面的代码也证实了其实 `finally` 会在第 4 行的 `return` 之前运行。然后我们再来看下面这段示例代码:
```Python
li = [1, 2]
def test():
    try:
        return li
    finally:
        li.append(3)
        
print(test())
```

打印的结果是什么？是的，是 [1, 2, 3] 。因为上文中我们已经说过了 `finally` 语句块会在 `return` 之前执行。那么我们再来看看下面这段代码:
```Python
str = "Helo World"
def test():
    try:
        str = "Hello Cat"
        return str
    finally:
        str = "Hello Python"

print(test())
```

打印的结果是什么？不是的，而是 Hello Cat ！不是说 finally 会在 return 之前运行吗？而且在上一个示例中， li 的值确实发生了变化啊。这是为什么？

因为在 return 之前，程序会暂存 str  的结果，之后去调用 finally 的语句。而 str 是  String 类型的，是值类型，所以是拷贝了一份，在 finally  中并的更改并没有生效。而之前的 li 变量之所以被更改了，是因为它是 List 类型的，在 Python 中， List 类型是引用类型，所以可以在 finally 中被更改。