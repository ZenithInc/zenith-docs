# 变量

这一篇文档，我们来描述如何去定义以及使用变量。Shell 中的变量大致分为自定义变量、位置变量以及环境变量。将在下文中分别加以描述。

### 自定义变量 {id="custom-variable"}

首先，来说一下变量命名的规则:

- 变量是由任何字母、数字和下划线组成的字符创，且不能以数字开头。
- 变量名称是严格区分大小写的，例如 Var1 和 var1 是不同的。
- 变量、等号、值中间不能出现任何空格。

变量如何定义和使用呢？
```shell
say='Hello World'
echo $say ## Output: Hello World
```
Shell 并不是一种十分严谨的脚本语言，不管你的变量类型是什么，对于 Shell 来说，都会作为字符串处理，比如下面这个例子:
```shell
num1=123
num2=456
echo $num1+$num2 ## 123+456
```
下面是一些错误的示例，在写脚本的过程中需要注意:
```shell
## 赋值语句等号的两边都不能有空格
count = 123  ## error
count= 123 ## error
count =123 ## error
## 变量名不能使用数字开头
1count=2	## error
```
另外，单个变量中可以存储多个值，并且使用数字作为索引进行遍历或单个引用，我们称之为数组。定义和使用如下:
```shell
arr_var=(a b c d e f g)
echo $arr_var  ## 输出第一个元素
echo ${arr_var[1]}  ## 输出第二个元素，索引从 0 开始
echo ${arr_var[*]}  ## 输出所有的元素: a b c d e f g
arr_var[3]=dd  ## 对制定索引重新赋值
echo ${arr_var[3]} ## 输出重新赋值之后的值: dd
```
> 需要注意的是，在 Ubuntu 中，默认使用的是 DASH 而不是 BASH, 以上数组语法会不支持。所以要执行脚本需要指定为 `bash script.sh` ，而不是 `sh script.sh` 。


### 位置变量 {id="position-variable"}

什么是位置变量呢？当一条命令或脚本执行时，后面可能会有多个传递的参数，而我们可以使用 Shell 内建的这些变量来引用这些参数。
```shell
cat p.sh ## Output:
#!/bin/bash
echo $0
echo $1
sh p.sh p1 ## Output:
p.sh  ## $0 表示第一个参数，即 Shell 文件名本身
p1 ## $1 表示文件名后面的第一个参数，$2 $3 $4 以此类推......
```
具体的位置变量如下表所示:

| 位置参数变量 | 含义                                                         |
|--------|------------------------------------------------------------|
| $n     | n 为数字，$0 代表脚本本身，$1 ~ $9 代表 1～9 个参数，10 以上的参数需要用大括号包含，如 ${10 |
| $@     | 命令行中所有的参数，但每个参数区别对待                                        |
| $*     | 命令行中所有的参数，所有参数视为一个整体                                       |
| $#     | 参数个数(不包括文件名本身)                                             |

这个位置变量会在执行函数的时候临时发生改变，如下示例:
```shell
#!/bin/bash

add()
{
  echo `expr $1 + $2` ## 在执行这个求和函数的过程中，$1 和 $2 临时指向了其函数内的传递的参数
}
add $1 $2
```
### 环境变量 {id="environment-variable"}

什么是环境变量呢？Linux 是一个多租户的操作系统，针对不同的用户都会有一个专有的运行环境，这些环境中的全局变量，我们就称之为环境变量。

环境变量根据其对用户的作用域可以划分为以下两种:

- 对所有用户生效的环境变量(一般存储在 `/etc/profile` 配置文件中)
- 对特定用户生效的环境变量(一般存储在 `~/.bashrc` 或者 `~/.bash_profile` 配置文件中)
- 临时有效的环境变量(在脚本或命令行使用 export 的时候)

有一些常用的环境变量，如下表所示:

| 环境变量     | 含义          |
|----------|-------------|
| PATH     | 命令搜索的路径     |
| HOME     | 用户家目录的路径    |
| LOGNAME  | 用户登录名       |
| PWD      | 当前所在路径      |
| HISTFILE | 历史命令的保存文件   |
| HISTSIZE | 历史命令保存的最大行数 |

```shell
## 当我们执行各种 Linux 命令的时候，其实就是一些已经写好的程序
## 这些命令分布在不同的目录下，如果需要在任意目录都可以执行这些程序
## 就需要将命令加入到 $PATH 这个环境变量中
echo $PATH ## Output: /usr/local/sbin:/usr/local/bin:/usr/sbin:......省略
```
> 注意: 约定俗成环境变量的名称全部使用大写，使用下划线(_)分隔。


如果需要查看所有的环境变量，我们可以使用 `printenv` 或者 `env` 命令，如下截图所示:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/E77uh60mVBqHOXJloJam.png)

可以创建一个环境变量，也可以删除一个环境变量:
```shell
export my_var=test
printenv my_var ## Output: test
unset my_var ## 删除自定义的环境变量
printenv my_var ## 什么也不会输出
```

### 变量替换 {id="variable-replace"}

变量替换就是指，通过既定的一些语法规则，对变量的内容进行特定的处理。比如说，字符串进行截取、匹配以及替换。主要的规则如下表所示:

| 语法                | 说明                          |
|:------------------|:----------------------------|
| ${变量名#匹配规则}       | 从变量开头进行规则匹配，将符合最短的数据删除      |
| ${变量名##匹配规则}      | 从变量开头进行规则匹配，将符合最长的数据删除      |
| ${变量名%匹配规则}       | 从变量尾部进行规则匹配，将符合最短的数据删除      |
| ${变量名%%匹配规则}      | 从变量尾部进行规则匹配，将符合最长的数据删除      |
| ${变量名/旧字符串/新字符串}  | 变量内容符合就字符串，则第一个就字符串会被新字符串取代 |
| ${变量名//旧字符串/新字符串} | 变量内容符合旧字符串，则全部的旧字符串会被新字符串取代 |

举例说明如下:
```bash
$ echo ${PATH}
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/bin
$ echo ${PATH#*bin}     ## 删除符合以 bin 开头的最短的字符串
:/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/bin
$ echo ${PATH##*bin}    ## 删除符合以 bin 开头的最长的字符串(因为 bin 结尾，所以未空)

$ echo ${PATH%bin*}   ## 和 # 的区别是，# 是从左到右进行匹配，而 % 是从右到左进行匹配
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/
$ echo ${PATH%%bin*}  ## 双百分号和单百分号的区别是，双百分好表示最长匹配，而单百分号则表示最短匹配
/usr/local/
```
然后，我们针对字符替换举例说明:
```bash
$ echo $PATH
## /usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/bin
$ echo ${PATH/:/,}
/usr/local/bin,/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/bin
$ echo ${PATH//:/,}
/usr/local/bin,/usr/bin,/usr/local/sbin,/usr/sbin,/home/vagrant/.local/bin,/home/vagrant/bin
```
从上面的示例中可以看到，替换也一样， `/` 和 `//` 的区别就在于，在符合匹配规则的情况下，是单个替换还是多个替换。

### 变量测试 {id="variable-test"}

变量测试在实际的场景中应用并不是很多，其对应的规则如下表所示:

| 变量配置方式           | str没有配置  | str为空字符串 | str已配置且非空 |
|------------------|----------|----------|-----------|
| var=${str-expr}  | var=expr | var=     | var=$str  |
| var=${str:-expr} | var=expr | var=expr | var=$str  |
| var=${str+expr}  | var=     | var=expr | var=expr  |
| var=${str:+expr} | var=     | var=     | var=expr  |
| var=${str=expr}  | var=expr | var=     | var=$str  |
| var=${str:=expr} | var=expr | var=expr | var=$str  |

表格中的 `expr` 指的是字符串。举例说明:
```bash
str=
echo ${str:-defualt_value} ## 输出什么？
```
`${str:-expr}` 对应着表格中的第二行，而 `str` 变量已经声明(配置) 了，但是是空字符串，所以对应着第二行第三列的规则，即 `expr（default_value)` 。

### 有类型变量 {id="variable-type"}

Shell 本身就是解释性的脚本语言，他的变量是弱类型的。但 Shell 也是支持对变量的类型进行声明的。

我们可以使用 `declare` 或者 `typeset` 命令来声明变量的类型，它们两者之间是等价的。下面的内容将以 `declare` 命令做演示。

`declare` 命令可以将变量声明为下表中的这些类型:

| 参数 | 含义                 |
|----|--------------------|
| -r | 将变量设置为只读           |
| -i | 将变量设置为整数           |
| -a | 将变量定义为数组           |
| -f | 显示此脚本前定义过的所有函数以及内容 |
| -F | 仅显示此脚本定义过的函数名      |
| -x | 将变量声明为环境变量         |

我们可以把变量设置为只读变量，如下示例:
```bash
$ declare -r x=3.14
$ x=3.14156
-bash: x: readonly variable  ## 提示变量为只读的，不允许修改
```
类似的，如果我把变量设置为了整数:
```bash
declare -i num=3
num=Hello
echo $num  ## 输出:0
declare -i count=10
count=$count+90
echo $count  ## 输出:100,如果不指定类型，shell 会将算数运算做字符串拼接处理
```
最后，在演示如何设置环境变量以及取消设置:
```bash
declare -x e=2.15   ## 设置
printenv | grep e=2.15 ## 输出: e=2.15
declare +x e=2.15   ## 取消设置
printenv | grep e=2.15 ## 什么也没有输出
```
### 全局变量和局部变量 {id="global-variable-and-local-variable"}

在 Shell 中如果不做特殊的声明，那么所有的变量都是全局变量。在大型的脚本程序中，谨慎使用全局变量。
```bash
#!/bin/bash

this_pid=$$
function test() {      
        echo $this_pid
        test_var='test'
}
test   ## 输出: 该脚本的进程id,是一个整数，例如 24967
echo $test_var  ## 输出: test
```
从上面的示例中可以看出，定义在函数外部的变量，在函数内部可以直接使用。而定义在函数内部的变量，在函数执行之后，也可以在函数外部再次使用。

如果不希望函数的内部定义的变量在外部使用，可以使用 `local`   关键字对变量的定义进行修饰:
```bash
function test() {
	local test_var='test'
}
echo $test_var  ## 输出为空
```
**如果函数内外存在同名的变量，则函数内部的变量会覆盖外部的变量。**
### 总结

本文主要讲解了 Shell 脚本编程中的变量，有那一些类型？如何定义以及如何使用。
