# 语法快速入门

这一系列的文档使用 Python3, 采用 Linux 环境编程。

### Hello World

我们先从 Hello World 开始:

```python
print("Hello World!")
```

### 定义变量

上面使用的 "Hello World" 是一个字符串，我们可以将这个字符串赋值给一个变量:

```python
message = "Hello World"
print(message)
```

### 变量的类型

我们的变量是有类型的？ **为什么需要有类型呢？因为这些变量的值都需要存储在内存中，不同类型的值在内存中的占用容量也不一样。所以我们定义了类型，以便在内存中提前为这些变量的值开辟空间** 。

Python 中支持的类型如下图所示:
![image.png](https://cdn.nlark.com/yuque/0/2020/png/502915/1596461913799-2573dacf-ad03-40b9-9955-10332a1a47cc.png#align=left&display=inline&height=547&originHeight=1094&originWidth=1714&size=356576&status=done&style=stroke&width=857)

#### 字符串和数值

最常见的类型莫过于字符串和数值了，在我们的生活中也无时不刻在用。比如我们去买菜，买了什么？白菜，白菜是一个名字，是一个字符串。这个白菜多少一斤呢？1块钱一斤，那么这个1块钱就是数值。

我们之前定义的 message 的变量就是一个字符串类型。不同的类型，会有不同的方法？什么是方法啊？就是预定义的操作。比如我们可以将字符串全部转成小写输出:

```python
message = "Hello World"
print(message.lower())  ## Hi, Bob
```

此外，我们还可以将多个字符串通过 `+` 拼接起来，就拿打招呼为例:

```python
name = "Bob"
print("Hi," + " " + name)
```

接着，如果我去买菜，花了5.8，然后递给买菜的大妈一张 10 元人民币，按照道理她应该找我 3.2 元。但是呢？我想要这两毛钱也没什么用，我就说凑个整吧:

```python
price = 5.8
print(round(price))
```

此外，我们还可以对数值进行运算:

```python
print(10 * 10)  ## 100
print(20 - 10)  ## 10
print(30 / 2)   ## 15
print(15 + 16)
```

#### 列表和元组

在实际的生活中，我们经常会对事物按照不同的维度进行归类分组。而程序最大的目的就是为了模拟现实，解决现实中的一些问题。对于一组的数据，在程序中，我们可以通过 **列表** 这个概念来表示:

```python
languages = ["PHP", "Python", "Java", "C"]
print(languages)   ## ['PHP', 'Python', 'Java', 'C']
```

我们可以访问这个列表中的每一个元素，通过下标, 即为列表中的每一个元素按顺序进行标号，从 0 开始:

```python
languages = ["PHP", "Python", "Java", "C++"]
print(languages[0], languages[3])            ## PHP C++
```

既然可以访问，就可以修改:

```python
languages = ["PHP", "Python", "Java", "C++"]
languages[2] = "C"
print(languages[2])  ## C
```

可以追加，可以删除:

```python
languages = ["PHP", "Python", "Java", "C++"]
# 在列表的末尾添加
languages.append("Swift")
# 在指定的位置添加
languages.insert(1, "Ruby")
languages.pop(1)
print(languages)        ## ['PHP', 'Python', 'Java', 'C++', 'Swift']
```

可以对列表进行排序:

```python
languages = ["PHP", "Python", "Java", "C++"]
languages.sort()           ## ['C++', 'Java', 'PHP', 'Python']
print(languages)
languages.reverse()        ## ['Python', 'PHP', 'Java', 'C++']
print(languages)
```

**元组和列表十分相似，最大的区别在于元组中元素的地址是不可变的** 。

```python
## 定义并计算元组长度
pers = ("Bob", "Foo", "June")
print(len(pers))  ## 3
```

如果你尝试对元组中的元素的值进行重写的话，会产生一个错误:

```python
pers = ("Bob", "Foo", "June")
pers[0] = "Wang"
```

运行上面的程序，会输出如下错误:

```
C:\Users\admin\PycharmProjects\test\venv\Scripts\python.exe C:/Users/admin/PycharmProjects/test/test.py
Traceback (most recent call last):
  File "C:/Users/admin/PycharmProjects/test/test.py", line 2, in <module>
    pers[0] = "Wang"
TypeError: 'tuple' object does not support item assignment

Process finished with exit code 1
```

#### 字典

在现实的生活中，总有一些人的名字会让我们感到尴尬，因为当中的某个字我不认识。这时候，我就需要去查字典, 比如这个名字：林茜。通过查字典我知道，这个茜字有两个含义两个读音:

- 茜(qian: 第四声)：茜草，多年生攀缘草本植物。茎方形，有倒刺。根黄红色，可提取染料，也可供药用。
- 茜(xi: 第一声): 音译用字。多用于外国女子名。

通过上面这个例子，大概能感觉到，字典是一种映射，为了方便查询。在 Python 中，也提供了字典类型，提供了从 key 到 value 的映射。

```python
words = {"qian": "茜草，多年生攀缘草本植物。茎方形，有倒刺。根黄红色，可提取染料，也可供药用", "xi": "音译用字。多用于外国女子名"}
print(words["qian"])  # 茜草，多年生攀缘草本植物。茎方形，有倒刺。根黄红色，可提取染料，也可供药用
print(words["xi"])  # 音译用字。多用于外国女子名
```

和列表一样，我们可以对字典中的键值对进行添加、修改以及删除操作:

```python
user = {"name": "admin", "age": 14, "email": "admin@end.wiki"}
# 修改年龄
user['age'] += 1
# 添加工作
user['job'] = "soft developer"
# 删除邮箱
del user['email']
print(user)   # {'name': 'admin', 'age': 15, 'job': 'soft developer'}
```

### 分支以及循环

编程语言中核心的概念并不多，其中最基本的是变量、分支、循环。我们的代码按照执行顺序可以分为三种结构，分别是顺序、分支以及循环。

所谓顺序，就是上一行代码执行完成之后执行下一行代码。所谓分支，就是对条件进行判断，然后执行对应的代码块。而循环，则是对某一段代码进行多次执行，根据其是否满足条件决定执行的次数。

#### 分支

我们的生活中处处都存在着抉择，每一次抉择，都会或多或少改变我们人生的走向。比如晚上我是吃饭还是吃面:

```python
food = "noodles"
if food == "noodles":
    print("走去兰州拉面")    ## 还是吃面吧
else:
    print("走去地沟油菜馆")
```

但是生活的难并不仅仅是在于没有选择的余地，有时候也会出现有太多的选择:

```python
needs = "我想要钱，我想要房子，我想要名望，我想要..."
if needs == "money":
    print("奋斗或偷盗")
elif needs == "house":
    print("租或者买")
elif needs == "popular":
    print("选秀或者扑街")
else:
	print("做梦")
```

#### 循环

使用 while 语句进行循环，并通过对数值变量的递增或递减控制循环的次数:

```python
i = 1
while i <= 5:
    print(i)
    i += 1
```

在循环中，也可以通过 `continue` 跳过某次循环，或者通过 `break` 接触循环。

对列表或元组进行循环:

```python
languages = ["PHP", "Python", "Java", "Swift"]
for language in languages:
    print(language)
```

对字典进行循环:

```python
user = {"name": "admin", "email": "admin@end.wiki", "country": "China", "Job": "soft developer"}
for k in user:
    print(k + "\t:" + user[k])
```

输出内容如下:

```
name	:admin
email	:admin@end.wiki
country	:China
Job	:soft developer
```

### 函数

函数是对一段代码的分装，一个函数是一个动作、操作。比如一个函数是一声问候:

```python
def greet():
    print("Hello")
greet()    # Hello
```

也可以指定一个人问候:

```python
def greet(name):
    print("Hello, " + name)
greet("Bob")   # Hello, Bob
```

为函数的传参指定一个默认值:

```python
def greet(name = "Bob"):
    print("Hello, " + name)
greet()    # Hello, Bob
```

调用函数的时候，也可以指定传参的名称，可以增加可读性:

```python
def greet(name = "Bob"):
   print("Hello, " + name)
greet(name = "Foo")   # Foo
```

函数可以有返回值:

```python
def sum(num1, num2):
    return num1 + num2
print(sum(1, 2))   # 3
```

也可以传递任意数量的参数:

```python
def sum(*numbers):
    total = 0
    for number in numbers:
        total += number
    return total

print(sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10))  # 55
```

### 模块(Module)

上面我们这些内容都是很基础的，三两行代码，所以放在一个文件里也是没什么问题。如果我们真得去写一个项目，代码量可能就不会是三两行，而是三四千行、万行、数十万行。这么多的代码，可不能放在一个文件里哦，不方便管理和维护。

那么，我们就会将代码分割成不同的部分，放在不同的文件中。这时候，我们就可以使用模块了，一个模块其实就是一个以 `.py` 结尾的 Python 代码文件。这个文件中，声明了一些可能会被其他模块用到的变量、函数以及类。比如我们申明如下这个模块（文件命令为 database.py，当中有一个 Database 的类, 用来操作数据库:

```python
class Database:
    def query(self):
        print('Query Database...')
class Mysql:
    def query(self):
        print('MySQL query Database...')
```

模块有了，那么如何在其他的代码文件中引入这个模块中的这个 Database 类呢?

```python
import database

db = database.Database()
database.query()  # Query Database...
```

如果我并不像引入这个 database 文件，只是向引入部分内容呢？

```python
from database import Mysql

mysql = Mysql()
mysql.query()    # MySQL query Database...
```

虽然，我们也可以使用 `from databases import *` 来导入 database 模块中所有的内容(变量、函数以及类)。但是不可以这么做，因为这样做的话，你在阅读代码的时候，不知道这个变量、函数或者类来自哪里？因为你引入的时候使用了 * 号这个通配符，并没有说明从这个文件中引入了什么。

### 包(Package)

如果说模块是对变量、函数以及类的封装，那么包就是对模块的封装。简单的说，包当中含有模块，而模块中包含类、函数以及变量。

之所以会有模块、包这样的概念，就是因为随着代码量的增大，需要对代码进行拆分管理。这非常好理解，就像人多了之后，需要对人进行拆分管理，比如亚洲、中国、浙江、杭州、西湖区、XX街道、XX小区、3幢、一单元、201室......为什么需要分的那么细？因为人太多了，不分不方便管理。 **分而治之是计算机中非常常见以及重要的思想** 。

那么怎么创建一个包呢？很简单，再一个目录中创建一个空的文件，命名为 `__init__.py` 即可。这么文件所在的目录就是一个包。

```shell
mkdir pack && touch pack/__init__.py
```

然后我们再这个包当中加入一个模块,命名为 `pack/data.py` ，模块中包含一个方法:

```shell
def greet():
    print('Hello')
```

然后，我们一个文件，名为 `app.py` , 文件内容如下:

```python
import pack.data

pack.data.greet()
```

也可以写成如下形式:

```python
from pack.data import greet

greet()
```

包当中还可以包含子包，和目录层级一致，比如包名是 `package.sub.module` ,那么路径名也应该是 `package/sub/module.py` 。

经过这样一个例子，你明白为什么 Python 可以将我们拆分开的代码重新组合再一起执行了吗？**当我们执行 `python app.py` 的时候，python 的解释器会去逐行解释代码，当发现代码以 `import` 或者 `from` 开头的时候，会根据包名和模块名找寻对应的目录下的代码文件，然后将对应的文件内容插入 `app.py` 中逐行执行。**

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/ykfMwM8GxRzCiWBqEiXx.png" alt="draw" />
