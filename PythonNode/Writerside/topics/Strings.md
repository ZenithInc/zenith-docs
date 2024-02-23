# 字符串详解

什么是字符串？其实就是字符，区分于其他的类型，比如说数字。下面展示的都是一些字符串:

```python
str1 = 'Hello World'
str2 = '123'
str3 = "Let's go to moon"
str4 = """
   窗前明月光
   怀是滴上霜
"""
str5 = '''
   Left's go to moon
'''
```

### 多种引号之间的区别

在 Python 中，使用引号来包裹字符串内容。可以使用单引号、双引号以及像上面 `str4` 和 `str5` 那样使用连续的三个单引号或三个双引号来表示多行文本。

> 注意：这些引号都应该在英文输入法下输入，都是英文的引号，而不是中文。


那么单引号和双引号有什么区别呢？就如变量 `str3` 一样，双引号可以包含转义字符，比如 `'` 。如果在单引号中出现转义字符，则需要使用转移符号 `\` 进行转义，否则程序将出错:

```python
str1 = 'Let's go to moon'   # 这样写是错误的
str2 = 'Let\'s go to moon'  # 使用转义符号转义是正确的
str3 = "Let's go to moon'   # 双引号不需要明确使用转义符号进行转义
```

通常这种情况，推荐使用双引号，看起来比较舒服，也无需关系转义字符导致的程序异常。

三引号又分为三个单引号和三个双引号，它们都是用来表示多行文本，并没有区别。也就是说，两种方式都可以使用。

多行文本用在，单行文本太长的情况下，或者需要换行的情况。比如我需要用一个字符串变量来包含一段 Python 代码:

```python
code = '''
from flask import Flask
app = Flask(__name__)
@app.route('/')
def index():
    return 'Hello World'
'''
# 引号并不要求单独写一行，但是习惯上单独写一行，较为直观。
```

这种情况下，就很适合使用多行文本。Python 会在每一行末尾加入 `\n` 转义字符，表示换行。

### 什么是转义字符

什么是转义字符？可以理解成是特殊的字符。那么它们特殊在哪里呢？

**其中一部分的转义字符是为了表示看不见的字符。**我们很自然的以为，在键盘上敲击的每一个键都可以在屏幕上显示出起对应的字符。但实际上有一些键同样输出了字符，但在屏幕上并没有显示。比如说我们上面中出现的 `\n` 换行符。虽然看不见，但必须使用一个字符告诉计算机，我回车了。
**
**还有一类是在编程语言中，已经被语言本身使用并表示特殊含义的字符。**比如说引号，在 Python 中使用一对引号来表示字符串。这时候如果在一对双引号中再次输入一个双引号，就会导致程序没有办法判断哪里是字符串的开始、哪里是字符串的结束。

还有一些其他的作用，但是没有必要去死记硬背。记住几个常用的就好了，比如说 `\n` 表示换行、 `\r` 表示回车、 `\t` 表示制表符(Tab键)。

转义符号 `\` 本身也是转义字符，如果我希望它能在字符串中输出，那么也应该对它进行转义，比如:

```python
print('转义符号: \\')
```

另外，我们也可以通过原始字符串(Raw String)的方式来表达:

```python
path = r'C:\Windows\system64\etc\derver\etc\hosts'
print(path)
```

### 字符串的格式化

关于字符串的格式化，在 Python 不同的版本中有不同的推荐写法，虽然都向下兼容，但是我们推荐使用更新的写法。下面将针对不同的版本的不同写法，加以介绍, 可以对比一下。

在 **Python2.x** 中，有四种字符串的格式化方式。先说第一种，通过 `%` 格式化, 示例如下:

```python
name = 'Python'
greet = 'hello, %s' % name
print(greet)  # 输出： hello, Python
```

那么如果我有多个值呢？上面的形式要稍微改变一下, 使用元组的形式:

```python
import sys
name = 'Python'
print('%s version %s' % (name, sys.version)) # Python version 3.8.5......省略......
```

还有一种使用对象的方式，可以指定名称:

```python
code = 500
message = 'server error'
print('error code is %(code)x error message is %(message)s' % {'code': code, 'message': message})
```

`%{}x` 中的 `x` 表示十六进制显示这一种可读性比较好，也避免了因为传参顺序的颠倒导致的难以发觉的错误。然后我们再来看第二种写法, 这是在 **Python3 **中引入的新式的写法:

```python
code = 500
message = 'server error'
print('error code is {} error message is {}'.format(code, message))
```

还是使用上面的例子，这种写法看上去简洁了很多。同样，这种写法也支持命名的形式:

```python
code = 500
message = 'server error'
print('error code is {code:x} error message is {message}'.format(code=code, message=message))
```

`{code:x}` 表示将 `code` 以转换成十六进制显示。在 **Python 3.6 中新增了一种称为字符串字面量插值的写法**:

```python
code = 500
message = 'server error'
print(f'code is {code:x}, message is {message}')
```

这样的写法是不是更加的简洁?另外，这种写法还对插值进行了增强，支持表达式:

```python
product = 'iPhone12 SE'
price = 580000  # 以分为单位
print(f'{product} price is {price / 100:0.2f}') # 转换成以元为单位，并保留两位数
```

顺带介绍一下，其实还有一种方法，了解一下即可:

```python
from string import Template
name = 'Bob'
age = 18
print(Template('Hello, my name is $name, i am $age years old').substitute(age=age, name=name))
```

### 字符串的运算

我们在生活中，经常会对数值进行运算，那么什么叫做对字符串运算呢？我举一个例子：

```
老板: 这个东西今天能做完吗？
我: 不能！
老板: 把不字去掉！
```

对，这就是字符串的运算之一，对字符串内容的剪切。除此之外，我们还经常回对字符串内容进行拼接、替换等操作。

字符串的拼接:

```python
print('Hello' + ' ' + 'World')
```

上面是字符串的“加法运算”，表示对字符串进行拼接。那么如果是乘法呢？

```python
print('Hello' * 3)  # HelloHelloHello
print('Hello' * 'World') # 语法错误
```

字符串(String)本质上是一串字符(Char)，是 Char 的列表。所以，我们也可以使用列表的形式来访问字符串:

```python
print('Hello'[0]) # H
print('Hello'[-1])  # o
print('Hello'[0:3]) # Hel
print('Hello'[0:-1]) # Hell
print('Hello'[1:])   # ello, 截取到末尾
```

