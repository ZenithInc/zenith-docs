# 命令替换

在 Shell 脚本编程中，经常会通过执行命令来获取相关的数据。那么如何在脚本中，获取命令的输出呢？

命令的输出是不能直接赋值给变量的, 那怎么办呢？这就需要引出文本的主角了： **命令替换** 。使用命令替换有下面给两种方法:

|     | 语法格式         |
|-----|--------------|
| 方法一 | `command`    |
| 方法二 | `$(command)` |


## 使用反引号字符 {id="unquote"}

反引号字符？就是在键盘 `esc` 键下面的那个按键 ``` 。

我们用一对```` 符号来包括一条命令，然后 Shell 的解释器会执行这条命令，并且将该命令的输出赋值给指定的变量。如下示例:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/ua1b4s4YZwShU76JK9YF.png)

在来举一个例子，我要获取系统中所有的用户并逐行打印出来：

```bash
#!/bin/bash

for user in `cat /etc/passwd | cut -d ":" -f 1`
do
  echo $user
done
```

## 使用$() {id="dollar"}

除了使用反引号字符外，还可以使用 `$()` 。因为反引号字符容易和单引号混淆，而且不支持嵌套，所以，更推荐使用 `$()` 来使用命令替换。
```bash
echo $(date +%Y年%m月%d日) ## 输出: 2020年06月02日
```
由于命令替换会创建一个子 Shell 来执行，所以当中无法使用当前脚本中的变量。在命令行提示符下使用路径 `./` 运行命令的话，也会创建出子 shell;要是运行命令的时候不加入路径，就不会创建子 shell。如果你使用的是内建命令的 shell 命令，并不涉及子 shell。

下面这个例子演示了嵌套的命令替换:
```bash
## 获取明年
echo $(($(date +%Y) + 1)) ## 输出: 2021
```

## 避免换行符被替换 {id="entry"}

我们先来看一个案例，我们输出当前目录中的每一个文件:
```shell
#!/bin/bash

for row in $(ls -al)
do
	echo $row
done
```
看起来没有问题，执行一下就发现问题了, 输出类似如下效果:
```shell
$ sh test.sh
total
16
drwxr-xr-x
```
输出的内容都是以空格作为分隔的，而我们期望的是使用换行符来分隔的，应该如下效果:
```shell
$ ls -al
total 16
drwxr-xr-x  5 jinzhi  staff   160  4  1 15:28 .
drwxr-xr-x  7 jinzhi  staff   224  3 28 15:06 ..
-rw-r--r--  1 jinzhi  staff    54  4  1 15:28 test.sh
```
这里就有一个小技巧了，把变量的引用使用引号包裹起来就好，更改后的脚本如下:
```shell
#!/bin/bash

for row in "$(ls -al)"		# 加上引号
do
	echo "$row"		# 加上引号
done
```
## 案例：自动拉起 Nginx 服务 {id="case"}

下面这段代码可以自动检测 Nginx 服务是否宕机，如果是则重新启动 Nginx 进程:
```bash
#!/bin/bash

nginx_process_num=$(ps -ef | grep nginx | grep -v grep | wc -l)
if [ $nginx_process_num -eq 0 ];then
  systemctl start nginx
fi
```

## 总结 {id="summary"}

在Shell脚本编程中，要获取命令执行结果并赋值给变量，可以使用命令替换技术。有两种方法实现这一功能：

1. 使用反引号字符 `` `command` ``：这种方式会执行括号内的命令，并将输出内容赋值给指定变量。例如，通过循环遍历 `/etc/passwd` 文件中每一行
的第一个字段来打印系统所有用户。

2. 使用 `$()` 结构：更推荐这种语法，因为它较不易与单引号混淆且支持嵌套。例如，`echo $(date +%Y年%m月%d日)` 输出当前日期，而
`$(($(date +%Y) + 1))` 则用于获取下一年的年份。

需要注意的是，命令替换会创建子 Shell 执行命令，因此在子 Shell 中无法直接访问当前脚本中的变量。若需避免换行符被替换成空格，在引用命令替换结果时
应加上引号，如 `for row in "$(ls -al)"`。

最后，提供了一个实际案例，展示了如何利用命令替换检测 Nginx 服务是否运行，如果进程数为0，则自动启动 Nginx 服务。