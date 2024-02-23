# 分支结构

我们写程序就是为了应对现实世界中的种种变化，既然有变化就需要有判断、选择以及分支。就像我们的人生一般，时时刻刻都面临着各种各样的选择，不同的选择决定了不同的人生走向。

在之前的文档中，已经出现过使用 `if...else...` 这样的分支语法了。所以，这一篇文档，我们就来详细描述在 Shell 编程中的分支结构。

## 使用 if...then 语法 {id="if-then"}

使用 `if...then` 的语法，我们需要制定判断的语句或者条件，如下是语句的示例:
```Shell
#!/bin/bash
if pwd ;then
	echo 'exec...'
fi
```

执行上面的脚本，会输出: `exec...` ，如果将 pwd 换成不存在的命令的话，分支内部的语句则不会执行。实际上，是根据命令的退出状态码来决定的，
如果是 0 值则会执行，非 0 值则不会执行。

## 使用 if-then-else 语句 {id="if-then-else"}

下面这个示例并没有实际的用处，只是用来演示`if-then-else`语句的用法，判断输入的第一个参数是否为 0:

```Shell
#!/bin/bash

if [ $1 -eq 0 ];then
  echo '0...'
else
  echo 'Not 0...'
fi
## 执行脚本
sh script.sh 0	## 输出: 0...
sh script.sh 1 ## 输出: Not 0...
```

## 多分支结构 {id="multiple-branches"}

接下来这个示例我们通过判断 Mysql 和 Nginx 的启动状态:

```Shell
#!/bin/bash

if ps -ef | grep mysqld | grep -v grep | grep -v sh &> /dev/null; then
	echo $?
	echo 'mysqld already start...'
elif ps -ef | grep nginx | grep -v grep | grep -v sh &> /dev/nul
then
	echo 'nginx already start...'
else
  echo 'failed...'
fi
```

先判断 mysql 是否已经启动，如果没有启动则接着判断 nginx 是否已经启动，如若都没有启动则输出failed...。

## 条件测试 {id="condition-testing"}

在 IF 的结构中，条件可以的执行一条命令，根据命令的退出状态码来决定是否执行。条件也可以是表达式，比如对数值、文本或者文件目录进行判断。

### 数值测试 {id="number-testing"}

上文中我们已经演示过数值条件判断了，比如 `[ $1 -eq 0 ]` 。其中 `-eq` 就是表达式中的比较符，表示等于的意思。更多的比较符如下表所示:

| 数值比较	       | 含义                               |
|-------------|----------------------------------|
| `n1 -eq n2` | 	n1 和 n2 相等，则返回 true,否则返回 false  |
| `n1 -ne n2` | 	n1 和 n2 不相等，则返回 true,否则返回 false |
| `n1 -gt n2` | 	n1 大于 n2,则返回 true, 否则返回 false   |
| `n2 -ge n2` | 	n1 大于等于 n2,则返回 true,否则返回 false  |
| `n1 -lt n2` | 	n1 小于 n2,则返回 true,否则返回 false    |
| `n1 -le n2` | 	n1 小于等于 n2,则返回 true,否则返回 false  |

然后我们写一个示例, 来比较给定的两个数值的大小:
```Shell
## 脚本内容如下
#!/bin/bash

if [ $1 -eq $2 ];then
	echo '$1 等于 $2'
elif [ $1 -gt $2 ];then
	echo '$1 大于 $2'
else
	echo '$1 小于 $2'
fi
## 执行脚本
bash script.sh 1 2 ## 输出:  $1 小于 $2
bash script.sh 1 1 ## 输出:  $1 等于 $2
bash script.sh 2 1 ## 输出:  $1 大于 $2
```

### 字符串条件测试 {id="string-testing"}

除了对数值进行比较外，我们也经常会对字符串进行比较，字符串的比较符如下图所示:

| 字符串比较	         | 含义                   |
|----------------|----------------------|
| `str1 = str2`  | 	相等比较                |
| `str1 != str2` | 	不等比较                |
| `str1 < str2`  | 	str1 小于 str2 为 true |
| `str1 > str2`  | 	str1 大于 str2 为 true |
| `-n str1`      | 	str1 长度不是 0 则为 true |
| `-z str1`      | 	str1 长度为 0 则为 true  |

下面的程序对比两个字符串是否相等，输出为“字符串不相等”：

```Shell
#!/bin/bash

str1='Hello'
str2='World'
if [ $str1 = $str2 ];then
	echo '字符串相等'
else
	echo '字符串不相等'
fi
```

> 需要注意: 如果对字符串进行大小的比较，由于 < 以及 > 会解析成重定向符号，所以需要转义，写成`[ $str1 \< $str2 ]`。

另外，如果是对字符串进行长度的判断，需要格外的小心：
```Shell
#!/bin/bash

if [ -n $str ];then  ## $str 在某些环境下会出错，写成 "$str"可以避免这种情况
	echo '字符串不为空'
else
	echo '字符串为空'
fi
```

### 文件目录测试 {id="file-dir-testing"}

接下来，我们来说说在 Shell 脚本中如何对文件目录进行测试。具体的比较符如下:

| 比较符	              | 含义                      |
|-------------------|-------------------------|
| `-d file`         | 	file 是否为目录             |
| `-f file`         | 	file 是否为文件             |
| `-e file`         | 	file 是否存在              |
| `-r file`         | 	file 是否可读              |
| `-w file`         | 	file 是否可写              |
| `-x file`         | 	file 是否可执行             |
| `-s file`         | 	file 存在则非空             |
| `file1 -nt file2` | 	file1 比 file2 新则为 true |
| `file1 -ot file2` | 	file1 比 file2 旧则为 true |

然后根据上表，我们来做一个示例:
```Shell
[ -d /etc/passwd ];echo $?  ## 输出: 1， 表示不是目录
[ -f /etc/passwd ];echo $?  ## 输出: 0， 表示是文件
[ -e /etc/passwd ];echo $?  ## 输出: 0， 表示文件存在
[ -r /etc/passwd ];echo $?  ## 输出: 0， 表示文件可读
[ -w /etc/passwd ];echo $?  ## 输出: 1， 表示文件不可写
[ -x /etc/passwd ];echo $?  ## 输出: 1， 表示文件不可执行
[ -s /etc/passwd ];echo $?  ## 输出: 0， 表示文件不为空
```

### 复合条件测试 {id="multiple-condition-testing"}

所谓复合条件，即对多个条件进行判断。多个条件之间的关系分为三种，分别为与或非。如下示例，判断一个文件可读并且可写:

```Shell
[ -r /etc/passwd ] && [ -w /etc/passwd ];echo $?  ## 输出为 1，表示文件不满足即可读又可写
[ -r /etc/passwd ] || [ -w /etc/passwd ];echo $?  ## 输出为 0，表示文件或者可读或者可写或者都满足
```

### 使用双括号作表达式的条件测试 {id="using-brackets-testing"}

双括号( `(())` )中间可以用来书写表达式作算术运算，支持的表达式如下表所示:

| 运算符        | 	含义   |
|------------|-------|
| `value++`  | 	后增   |
| `value--`	 | 后减    |
| `++value`	 | 先增    |
| `--value`	 | 先减    |
| `!`        | 	逻辑求反 |
| `==`       | 	相等   |
| `>`	       | 大于    |
| `<`	       | 小于    |
| `>=`       | 	大于等于 |
| `<=`       | 	小于等于 |
| `&&`       | 	逻辑与  |
| `          |       |`       |	逻辑或|

在使用双括号表达式的时候，需要注意以下内容:

* 双括号结构值嗯，变量名引用可以加 $ ，也可以不加
* 运算符前后可以有空格，也可以没有
* 多个运算符使用逗号分隔

用一个示例来演示:
```Shell
## 使用方括号的写法
[ $1 -gt $2 ]  ## 中括号前后必须要有空格，否则语法错误
(($1 > $2))  ## 括号前后可以有空格，也可以没有空格

## 两个数值比较大小
if (( $1 > $2 || $3 == $2 ));then
	echo '大于'
else
	echo '小于'
fi
```

### 使用双方括号作表达式的条件测试 {id="using-square-brackets-testing"}

和双括号类似，双方括号也可以包含类似 C  语言形式的表达式，如下示例:
```Shell
#!/bin/bash

a=1
b=2
c=3
## 使用单个方括号的写法
if [ $a -gt $b ] && [ $b -lt $c ];then
	echo 'Hello World'
fi
## 使用多个方括号的写法
if [[ $a -gt $b && $b -lt $c ]];then
	echo 'Hello World'
fi
```

需要注意的是:

* 在双方括号中，变量必须使用 $ 引用。 
* 和双括号一样的是，双括号前后必须要有空格，但比较符前后可以没有空格，比如 `[[ $a<$b || $b==$c ]]` 。

## 使用 case {id="case"}

当分支特别多的时候，我们可以使用 case 命令来表达:
```Shell
#!/bin/bash

case $1 in
	jack)
		echo 'Hello, jack'
		;;
	mike)
		echo 'Hello, mike'
		;;
	*)
		echo 'Hello, everyone'
		;;
esac
```

然后我们执行这个脚本:

```Shell
sh script.sh ## 输出: Hello, everyone
sh script.sh jack ## 输出: Hello, jack
sh script.sh mike ## 输出: Hello, mike
```

## 总结 {id="summary"}

本文详细描述了在 Shell 编程中分支结构的应用，以及其对应的注意事项。