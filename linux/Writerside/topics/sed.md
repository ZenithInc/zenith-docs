# Sed 流编辑器

什么是 sed? 官方的说法如下:

> sed is commonly used to filter text, i.e., it takes text input, performs some operation (or set of operations) on it, and outputs the modified text. sed is typically used for extracting part of a file using pattern matching or substituting multiple occurrences of a string within a file.

sed 通常用作过滤文本，也就是针对输入的一段文本对其做一些操作，然后输出经过操作已经变更的文本内容。sed 典型的应用是使用模式匹配提取文件中的一部分，或者替换文件中多次输出的字符串。

什么是流编辑器呢？在 sed 的文档中，有如下描述:

> A stream editor is used to perform basic text transformations on an input stream (a file or input from a pipeline). 

一个流编辑器用作对输入流(来源于文件或管道的输入)进行基本的文本转换。换句话说，就是 **对标准输出或文件逐行进行处理** 。

说了那么多，套用官方文档的一个例子来解释上面的描述:
```bash
## 打印 input.txt
$ cat input.txt ## 输出内容如下
hello php
hello java
## 使用 sed
$ sed 's/hello/hi/' input.txt > output.txt
## 打印 output.txt
$ cat output.txt  ## 输出内容如下
hi php
hi java
```
我们可以通过 sed 去遍历 input.txt 文件中的每一行，然后将其中的 hello 替换成 hi,最后输出到 output.txt 文件中。

sed 有两种形式的语法:

- 第一种形式: stdout | sed [option] "pattern command"
- 第二种形式: sed [option] "pattern command" file

总的来说，sed 遍历标准输出中的每一行文本内容，使用给定的 pattern 进行匹配，符合 pattern 的内容使用给定的 command 进行处理。如果没有给定 pattern ，则每一行都应用 command。

## sed 的选项 {id="options"}

sed 常用的选项如下表所示：

| 选项 | 含义                   |
|----|----------------------|
| -n | 只打印模式匹配行             |
| -e | 直接在命令行进行 sed 编辑，默认选项 |
| -f | 编辑动作保存在文件中，指定文件执行    |
| -r | 支持扩展正则表达式            |
| -i | 直接修改文件内容             |

创建一个文件，命名为 sed.txt：
```
Hello PHP
Hello Shell
Hello C
```
然后，我们使用默认模式( `-e` )对 sed.txt 进行处理:
```bash
$ sed "p" sed.txt ## 执行这条命令，p 是内置的命令表示为打印，输出内容如下
Hello PHP
Hello PHP
Hello Shell
Hello Shell
Hello C
Hello C
```
这就是默认模式，会将匹配的内容和修改之后的内容都输出到终端。如果我们希望改变这种模式，只输出修改后的内容，可以使用 `-n` 选项:
```bash
$ sed -n "p" sed.txt ## 输出内容如下
Hello PHP
Hello Shell
Hello C
```
如果是一个特别大的文本，使用默认模式将所有行都打印出来，会造成干扰。而使用 `-n` 选项，可以只打印模式匹配的行。通过这个例子，我们就可以了解到这些命令选项的基本用法。

接下来，我们再看看 `-e` 选项。sed 命令后面可以加上任意数量的 `-e` 选项，用来匹配更多模式的文本，比如我想要匹配 PHP 和 Shell：
```bash
$ sed -n -e '/PHP/p' -e '/Shell/p' sed.txt ## 输出内容如下
Hello PHP
Hello Shell
```
如果只有一个匹配模式的话，可以不用指定 `-e` 选项，但如上示例中需要指定多个匹配模式的情况下，就需要显式的使用 `-e` 选项。

`-f` 选项可以指定一个包含匹配模式和处理命令的文本，当这部分内容特别复杂的时候。

而 `-r` 指的是，匹配模式字符串支持正则表达式。比如上面的例子，我们使用正则来改写:
```bash
$ sed -nr '/PHP|Shell/p' sed.txt ## 输出内容如下
Hello PHP
Hello Shell
```

而 `-i` 表示将匹配并操作后的内容直接写入源文件,如下示例:
```bash
$ sed -i 's/Hello/Hi/g' sed.txt
$ cat sed.txt  ## 我们看到源文件的内容已经被修改了
Hi PHP
Hi Shell
Hi C
```
## sed 的 pattern

sed 是通过对文本流逐行进行模式的匹配，然后应用命令，从而来达到对文本流的转换。那 sed 都有哪些常用的 pattern 模式呢? 如下表所示:

| 匹配模式                         | 含义                                    |
|------------------------------|---------------------------------------|
| 10command                    | 匹配到第10行                               |
| 10,20command                 | 匹配从第10行开始，到第20行结束                     |
| 10,+5command                 | 匹配从第10行开始，到第16行结束                     |
| /pattern1/command            | 匹配到 pattern1 的行                       |
| /pattern1/,/pattern2/command | 匹配从满足 pattern1 的行开始，到满足 pattern2 的行结束 |
| 10,/pattern1/command         | 匹配从第10行开始，到匹配到 pattern1 的行结束          |
| /pattern1/,10command         | 匹配从满足 pattern1 的行开始，到第10行结束           |

针对上面的这些匹配模式，示例如下:
```bash
$ sed -n '2p' /etc/passwd 			## 打印第二行
bin:x:1:1:bin:/bin:/sbin/nologin

$ sed -n '2,3p' /etc/passwd   	## 打印第二到第三行
bin:x:1:1:bin:/bin:/sbin/nologin       
daemon:x:2:2:daemon:/sbin:/sbin/nologin

$ sed -n '2,+2p' /etc/passwd    ## 打印第二到第四行
bin:x:1:1:bin:/bin:/sbin/nologin       
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin

$ sed -n '/root/p' /etc/passwd  ## 打印匹配 root 的行
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin

$ sed -n '/halt/,/games/p' /etc/passwd  ## 打印匹配 hatl 到 匹配 games 之间的行
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin  
games:x:12:100:games:/usr/games:/sbin/nologin

$ sed -n '3,/adm/p' /etc/passwd					## 打印从第三行开始，到匹配 adm 的行结束
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin

$ sed -n '/adm/,5p' /etc/passwd				## 打印从匹配 adm 的行开始，到第 5 行结束
adm:x:3:4:adm:/var/adm:/sbin/nologin    
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

## sed 中的命令 {id="commands"}

前文中讲解了 sed 中的 pattern，接着我们再来看看 sed 中的各类命令，无外乎是增上改查，如下表所示:

| 类别 | 命令           | 含义                      |
|:--:|--------------|-------------------------|
| 查询 | p            | 打印                      |
| 增加 | a            | 行后增加                    |
|    | i            | 行前增加                    |
|    | r            | 外部文件读入，行后追加             |
|    | w            | 匹配行写入外部文件               |
| 删除 | d            | 删除                      |
| 修改 | s/old/new    | 将行内第一个 old 替换成 new      |
|    | s/old/new/g  | 将行内全部的 old 替换成 new      |
|    | s/old/new/2g | 将行内前 2 个 old 替换成 new    |
|    | s/old/new/ig | 将行内 old 全部替换成 new,忽略大小写 |

为了接下去的演示，我们先创建一个 vim 的配置文件，位于 `~/.vimrc` ,内容如下:
```
syntax on
set nu
set ts=2
```
### 删除 {id="delete"}

现在来演示删除命令:
```bash
$ sed -i '/syntax/d' ~/.vimrc   ## 删除匹配 syntax 的一行配置
$ cat ~/.vimrc  ## 输出内容，可见已经被删除了
set nu
set ts=2
```
### 新增 {id="insert"}

在删除了之后，我们恢复这行配置:
```bash
$ sed -i '/nu/a syntax\ on' ~/.vimrc   ## 空格需要使用转义字符(\)转义
$ cat ~/.vimrc  ## 输出内容如下，在匹配 nu 的后面一行增加了 syntax on 配置
set nu
syntax on
set ts=2
```
上面演示了命令 `a` 的用法，命令 `i` 也是类似，前者是在匹配行的行后增加，后者是在行前增加。然后我们演示如何使用 `-r` 命令，从指定的文件追加内容:
```bash
## 首先创建一个配置文件
cat <<EOF > config.txt
> set showcmd
> set encoding=utf-8
> set t_Co=256
> EOF
## 然后将这个文件追到到 ~/.vimrc 中
sed -i '/ts/r config.txt' ~/.vimrc
## 再来看看现在 ~/.vimrc 中的内容
$ cat ~/.vimrc  ## 输出内容如下
set nu
syntax on
set ts=2
set showcmd
set encoding=utf-8     
set t_Co=256
```
命令 `w` 是将匹配的行写入到指定的文件中:
```bash
$ sed -n '/ts/,/Co/w /tmp/result.txt' ~/.vimrc    ## 将符合 ts 匹配的行到符合 Co 匹配的行之间的内容输出到 /tmp/result.txt 文件中
$ cat /tmp/result.txt  ## 输出内容如下
set ts=2
set showcmd
set encoding=utf-8
set t_Co=256
```

### 更新 {id="update"}

然后，我们在来演示一下更新:
```bash
## 将符合文本 ts=2 换成 ts=4,注: 这表示每个制表符(tab)等于 n 个空格
$ sed -i 's/ts=2/ts=4/' ~/.vimrc
## 上面的示例是将第一个匹配的修改，如果需要全局的话，如下
$ sed -i 's/ts=2/ts=4/g' ~/.vimrc
## 如果需要忽略大小写
$ sed -i 's/ts=2/ts=4/i' ~/.vimrc
## 如果需要全局忽略大小写的话
$ sed -i 's/ts=2/ts=4/gi' ~/.vimrc
```
需要注意，上面说的全局的范围是文本流的行，而不是整个文本流。比如说  `sed -i 's/ts=2/ts=4/2g' ~/.vimrc` 表示一行中替换前两个匹配的字符串。

再来看一个例子，如果我需要将文本中的 `nu` 这个缩写换成完整的单词 `number` , 可以使用 **反向引用** ，即引用我匹配到的文本:
```bash
 ## & 就表示反向引用，这里指的是引用了 nu,在实际的引用场景中，往往是对前面正则匹配的文本的引用
$ sed -i 's/nu/&mber/g' ~/.vimrc
$ cat ~/.vimrc | grep nu  ## 输出: set number
```
> 注意: 在之前的示例中，我们使用的都是单引号，也可以使用双引号，区别在于双引号会去解析变量。

## 总结 {id="summary"}

本文详细介绍了 Unix/Linux 系统中强大的流编辑器 sed。sed 通过逐行处理文本流，根据指定的模式进行匹配，并执行相应的命令对文本进行修改、替换或增
删等操作。它有两种主要的语法形式：一种是针对标准输出，另一种是对指定文件进行操作。

在sed中，常用的选项包括`-n`（仅打印模式匹配行）、`-e`（执行多条编辑指令，默认选项）、`-f`（从文件读取编辑指令）、`-r`（支持扩展正则表达式）
和`-i`（直接修改原文件内容）。示例中展示了如何使用`-n`选项仅输出匹配到的行，并演示了多个匹配模式时如何结合`-e`选项来实现。

此外，文章详述了 sed 中的各种 pattern 匹配模式，如行号、行范围以及基于正则表达式的匹配，并通过实例展示了这些模式的具体应用。同时，还列举了 
sed 中的核心命令，包括查询命令（如`p`打印匹配行）、增加命令（如`a`追加到匹配行后方，`i`插入到匹配行前方，`r`从外部文件读入并追加，`w`将匹配
行写入外部文件）、删除命令（如`d`删除匹配行）和修改命令（如`s`用于替换字符串，可控制替换次数和是否忽略大小写）。

文中以创建和编辑`.vimrc`配置文件为例，生动地演示了 sed 的删除、新增、替换及追加外部文件内容等实用功能，并强调了反向引用在替换命令中的重要作用
。最后说明了单引号和双引号在编写sed命令时的区别，即双引号会解析内部的变量。总之，本文全面而深入地介绍了sed工具的使用方法及其在文本处理中的强大能力。