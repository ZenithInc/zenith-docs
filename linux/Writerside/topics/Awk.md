# Awk

什么是 awk 呢？awk 是一个文本处理工具，通常用于处理数据并生成结果报告。awk 和 sed 在工作形式上是类似的，都是通过对文本流进行逐行匹配进而施加
操作，只是 awk 可以在输出内容的前后在加上一些内容，比如页头和页尾。

它的基础语法如下所示:
```bash
awk 'BEGIN{}pattern{command}END{}' file_name
```
`BEGIN{}` 部分是对文本进行处理之前要进行的操作，而 `END{}` 则是之后。中间部分 `pattern{command}` 则和 sed 是一致的，通过对每一行进行匹配
，对匹配的内容使用 command 进行处理。

**awk 程序的核心思想是模式(pattern)/行为(action)这对概念，也叫模式驱动编程** ——《[Linux就是这个范儿](https://book.douban.com/subject/25918029/)》。

实际上， awk 也是一种图灵完备的微信语言。其实现了变量、分支以及循环等语言特性。其工作流程如下图所示:

![image.png](https://cdn.nlark.com/yuque/0/2020/png/502915/1591597745195-8ec0847e-d07f-4028-80b0-8e88ed40b32a.png#height=427&id=s7o17&originHeight=853&originWidth=1195&originalType=binary&ratio=1&rotation=0&showTitle=false&size=129352&status=done&style=none&title=&width=598)


## awk 的内置变量 {id="build-in-variables"}

上面提到了 awk 是一种图灵完备的微型语言，它可以自定义变量，也拥有很多内置的变量。这些内置的变量会在 awk 的编程中经常使用到，如下表所示:

| 内置变量     | 含义                        |
|----------|---------------------------|
| $0       | 整行内容                      |
| $1-$n    | 当前行的第1-n个字段               |
| NF       | 当前行的字段个数，也就是有多少列          |
| NR       | 当前行的行号，从1开始计数             |
| FNR      | 多文件处理时，每个文件行号单独计数，都从0开始   |
| FS       | 输入字段分隔符。不指定默认以空格或 tab 键分割 |
| RS       | 输入行分隔符。默认回车换行。            |
| OFS      | 输出字段分隔符。默认为空格。            |
| ORS      | 输出行分隔符。默认为回车换行。           |
| FILENAME | 输出文件名                     |
| ARGC     | 命令行参数个数                   |
| ARGV     | 命令行参数数组                   |

结合上面的变量以及 awk 的语法，写第一个示例,输出系统中的所有用户:
```bash
## 不使用 awk
$ cat /etc/passwd | cut -d: -F1
## 使用 awk
$ awk 'BEGIN{FS=":"}{print $1}' /etc/passwd  ## 如果是输出最后一个字段呢？使用 $NF 变量
## 同样的，我们也可以使用 -F 参数来指定分隔符
$ awk -F: '{print $1}' /etc/passwd
```
在默认情况下，awk 是使用空格或者制表符(tab键)作为分隔符的。在上面的例子中，因为 `/etc/passwd` 这个配置文件使用冒号作为分隔符，所以我们
在处理之前使用 `BEGIN{FS=":"}` 指定分隔符。然后我们使用内置的变量 `$1` 打印第一字段。

下面这个例子，展示了如何连同输出文件的行号:
```bash
$ awk 'BEGIN={FS=":"}{print NR"-"$1}' /etc/passwd
1-root
2-bin
......省略
```
除了 `NR` 之外，还有一个 `FNR` ,他们之间的区别在于处理多个文件的是时候，后者会对每个文件的行独立编号。而 `NR` 则是，合并编号，
如果 A 文件三行，B 文件二行，输出就是 1,2,3,4,5。

下面这个例子，演示了行分隔符:
```bash
$ cat javascript.js
var a = 1;b  = 2;console.log(a + b)

$ awk 'BEGIN{RS=";"}{print $0}' javascript.js
var a = 1
b  = 2
console.log(a + b)
```
另外，我也可以改变输出的字段分隔符，如果是行分隔符也是同理:
```bash
$ awk 'BEGIN{FS=":";OFS="-"}{print $1,$7}' /etc/passwd
root-/bin/bash
bin-/sbin/nologin
daemon-/sbin/nologin
......省略
```

## 格式化输出 {id="format"}

在 C 语言中，我们通常使用 `printf` 来格式化输出字符串，awk 也是一样的, 其支持的格式符下表所示:

| 格式符 | 含义            |
|-----|---------------|
| %s  | 打印字符串         |
| %d  | 打印十进制数        |
| %f  | 打印一个浮点数       |
| %x  | 打印十六进制数       |
| %o  | 打印八进制数        |
| %e  | 打印数字的科学计数法形式  |
| %c  | 打印单个字符的ASCII码 |

此外，还支持以下几种修饰符:

| 修饰符 | 含义                      |
|-----|-------------------------|
| -   | 左对齐                     |
| +   | 右对齐                     |
| #   | 显示8进制在前面加0，显示16进制在前面加0x |

下面这个例子展示了使用格式符打印用户名和用户的家目录:
```bash
$ awk 'BEGIN{FS=":"}{printf "Username is %s, Home at %s\n", $1, $NF}' /etc/passwd
Username is root, Home at /bin/bash
Username is bin, Home at /sbin/nologin
Username is daemon, Home at /sbin/nologin
......省略
```
同样是上面这个例子，我们可以限定每一列的长度以及使用修饰符指定是左对齐(默认是右对齐)：
```shell
$ awk 'BEGIN{FS=":"}{printf "%-20s%-20s\n", $1, $NF}' /etc/passwd
root                /bin/bash
bin                 /sbin/nologin
daemon              /sbin/nologin
adm                 /sbin/nologin
```
`print` 和 `printf` 只是 awk 众多字符串处理函数中的一个，下表列出了其他一些常用的字符串处理函数:

| 函数定义                              | 说明                                                          |
|-----------------------------------|-------------------------------------------------------------|
| `length(string)`                  | 返回字符串的长度                                                    |
| `substr(string,start,len)`        | 提取子字符串                                                      |
| `tolower(string)`                 | 将字符串全部转换为小写字母                                               |
| `toupper(string)`                 | 将字符串全部转换为大写字母                                               |
| `index(string,find)`              | 字符串查找，找到返回 find 在 string 中的起始位置，否则返回0                       |
| `match(string,regexp)`            | 与 index 类似，不同的是支持正则表达式                                      |
| `sub(regexp,replacement,target)`  | 字符串替换。将 target 与正则表达式 regexp 进行匹配，将第一个匹配到的结果替换成 replacement |
| `gsub(regexp,replacement,target)` | 与sub类似，区别是将全部匹配结果进行替换                                       |
| `split(string,array,regexp)`      | 分割字符串，分隔符使用正则表达式表示                                          |
| `sprintf(format,...)`             | 返回格式化的字符串，与 C 的 sprintf 类似                                  |
| `printf(format,...)`              | 输出格式化的字符串，与 C 的printf类似                                     |


## 模式匹配 {id="pattern-matching"}

前面说到，awk 是模式驱动的，在官方文档中也称之为是数据驱动的。因此，它更像是一个数据库，sed 是对每一行进行处理，而 awk 还可以对每一行中指定的
字段进行分隔、匹配以及处理。

awk 支持一下两种模式进行匹配:

- RegExp 按正则表达式匹配
- 关系运算匹配，比如 $1 >  50，类似于数据库中的范围查询

第一种，基于 RegExp 按照正则表达式匹配。这和 Grep 十分想象，甚至可以做到比 Grep 更加的强大，如下示例:
```bash
awk '/root/{print $1}' /etc/passwd
```
第二种，基于关系运算符匹配。支持的关系运算符如下表所示:

| 关系运算符 | 描述       |
|-------|----------|
| <     | 小于       |
| >     | 大于       |
| <=    | 小于等于     |
| >=    | 大于等于     |
| ==    | 等于       |
| !=    | 不等于      |
| ~     | 匹配正则表达式  |
| !~    | 不匹配正则表达式 |

如果涉及到多个条件，救下要下表中的这些布尔运算符了:

| 运算符          | 描述 |
|--------------|----|
| &#124;&#124; | 或  |
| &&           | 与  |
| !            | 非  |

下面这个例子，找出文本中某个数值大于 19 的行:
```bash
$ cat demo.txt
A 10 95
B 20 62
C 30 20
## 单个条件的匹配
$ awk '$2 > 19{print $0}' demo.txt
B 20 62
C 30 20
## 多个条件的匹配
awk '$2 > 19 && $3 >20{print $0}' demo.txt
B 20 62
```
下面演示如何使用正则表达式进行匹配:
```bash
$ awk -F: '$1~/root/{print $0}' /etc/passwd  ## 对第一个字段进行正则匹配，不匹配使用 !~ 符号
root:x:0:0:root:/root:/bin/bash
```

## 表达式 {id="expression"}

awk 支持的表达式如下表所示:

| 运算符          | 含义             |
|--------------|----------------|
| +、-、*、/、%    | 加、减、乘、除、取模     |
| ^或**         | 乘方             |
| ++x          | 在返回x变量之前，x变量加1 |
| x++          | 在返回x变量之后，x变量加1 |
| +=、*=、**=、/= | 类似C语言的，简写形式    |

针对上面的运算符，示例如下：
```bash
## 加等于
$ awk 'BEGIN{num1=10;num2+=num1;print num1,num2}'
10 10
## 取模
$ awk 'BEGIN{num1=10;num2=num1/3;print num1,num2}'
10 3.33333
## ++
$ awk 'BEGIN{num1=10;++num1;print num1}'
```
如果要统计一个文件的空白行数量，可以使用 `grep` ,也可以使用 `awk` ：
```bash
$ grep '^$' /etc/service | wc -l
$ awk '/^$/{count++}END{print count}' /etc/service
```
遍历文件中的每一行，符合正则 `/^$` 模式的，执行操作 `{count++}` ,最后 `END` 执行变量的打印 `print count` 。下面这个例子，
用来求平均分:
```bash
$ cat demo.txt
Allen 80 90 96 98
Mike 93 98 92 91

$ awk '{print $1, $2, $3, $4, $5, ($2+$3+$4+$5)/4}' score.txt ## 使用括号提升运算优先级
Allen 80 90 96 98 91       
Mike 93 98 92 91 93.5
```

## 条件分支 {id="conditions"}

在 awk 编程中，分支语句的语法和 C 语言是非常类似的。我们来看一个例子，我要从下面这个文件中找出价格大于 100 并且小于 200 的产品，并打印对应的
产品名和价格。示例文件如下:
```shell
$ cat products.txt
Apple 100
Banana 120
Orange 90
Tea 180
Bread 200
```
实现程序如下:
```shell
$ awk '{if ($2 > 100 && $2 < 200) print $1"-"$2}' products.txt
Banana-120
Tea-180
```
由于代码比较多，所以我们把代码写到一个名为	`filter.awk`的文件中，可读性比较好:
```shell
$ cat filter.awk
{
	if ($2 > 100 && $2 < 200) {
		print $1"-"$2
	} else if ($2 < 100) {
		 print $1 " less 100"
	}
}
```
执行这个文件中的脚本:
```shell
$ awk -f filter.awk products.txt
Banana-120
Orange less 100
Tea-180
```

## 循环 {id="loop"}

在 awk 语法中，支持 `for` 和 `while` 以及 `do...while...` 语法，我们使用这三种语法来编写 100 以内的求和。先使用`for`演示如下:
```shell
BEGIN {
	sum = 0
	for (count=0;count <= 100; count++) {
		sum += count
	}
	print "sum:"sum
}
```
> 因为没有处理文件，所以代码都需要写在 BEGIN  语句块中。

接着我们来演示使用`while`语句来实现上面的求和:
```shell
BEGIN {
	sum = count = 0
	while (count <= 100) {
		count++
		sum += count
	}
	print "sum:"sum
}
```
最后来演示使用`do...while...`实现上面的求和，实际上语法和 C 语言都是一样的, 所以不需要特别记忆:
```shell
BEGIN {
	sum = count = 0
	do {
		count++
		sum += count
	} while (count <= 100)
	print "sum:"sum
}
```

## 实践案例 {id="case"}

如果你需要关闭所有运行中的 vagrant 控制的虚拟机:
```shell
vagrant global-status | awk '$4 == "running"{print $1}' | xargs -p vagrant halt
```

## 总结 {id="summary"}

awk 是一个强大的文本处理工具，它以模式(pattern)/动作(action)为核心概念进行数据驱动编程。awk 通过逐行读取和处理文本流，并能在输出前后添加内容
，具备变量定义、条件分支、循环等高级语言特性，是一个图灵完备的微型语言。

awk 的基础语法结构包括 `BEGIN`、`pattern{command}` 和 `END` 三个部分。`BEGIN` 块用于在处理文件内容之前执行初始化操作，`END` 块则在处理
完所有输入后执行，而中间部分则是根据每行内容与指定模式匹配并执行相应的命令。

awk 内置了丰富的变量，如 `$0` 表示整行内容，`$1-$n` 分别代表每行的第1到第n个字段，`NF` 表示当前行的字段数，`NR` 为当前行号，`FNR` 在多文
件处理时独立计数每个文件的行号，以及控制输入输出分隔符的 `FS`、`RS`、`OFS` 和 `ORS` 等。awk 还支持自定义变量，使得用户能够灵活地处理和格式
化文本数据。

awk 提供了类似C语言中的 `printf` 函数来格式化输出，并支持一系列字符串处理函数如 `length()`、`substr()`、`tolower()` 等。此外，awk 支持
正则表达式和关系运算符来进行模式匹配，允许用户基于字段值执行复杂筛选和操作。

awk 具备条件分支语句，支持多种布尔运算符（如逻辑与`&&`、逻辑或`||`、逻辑非`!`）以及算术运算符，可以实现复杂的逻辑判断和数值计算。awk 还提供了
循环结构如 `for`、`while` 和 `do...while`，方便对数据集进行迭代处理。

实际应用中，awk 可以高效地完成各种任务，如统计分析、格式转换、提取特定信息等。例如，在 `/etc/passwd` 文件中查找用户名及其家目录信息，或者从
产品列表中筛选价格范围内的商品，都可通过 awk 轻松实现。此外，awk 还能与其他命令结合使用，例如在关闭 vagrant 控制的所有运行中虚拟机时，awk 
能够有效地辅助过滤出需要操作的目标。总之，awk 是一种强大且灵活的文本处理工具，广泛应用于脚本编写、系统管理、数据分析等诸多领域。