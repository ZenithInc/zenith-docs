# 函数的应用

对于一些重复的逻辑，我们需要将其封装成函数，有利于方便的调用以及后期统一的修改。合理的组织脚本的结构，可以拥有更好的可读性以及更高的可维护性。所以，我们来聊聊函数在 Shell 脚本编程中的应用。

## 函数的定义 {id="defined"}

在 Shell 中，函数有两种定义的语法，分别如下:
```shell
hello() {
  echo "hello, $1"
}
hello 'foo'
```
另一种是使用 function 这个关键字:
```bash
function hello() {
	echo "hello, $1"
}
hello 'foo'
```
对于函数的调用都是一样的，和普通的命令的使用是一致的。而函数的传参，直接在函数内部使用 `$n` 的方式来引用。

比如，我们来实现一个 `help` 的函数:
```bash
help() {
        echo 'usage: bc [options][file ...]'
        echo -e '\t-h  --help'
        echo -e '\t-i  --interactive'       
        echo -e '\t-l  --mathlib'
        echo -e '\t-q  --quiet'
}
help
```

## 参数的传递 {id="args"}

参数的传递在上面的例子中我们已经演示了，然后我们在来写一个计算器的例子演示:
```shell
#!/bin/bash

calc() {
	case $3 in
		\+)
			echo `expr $1 + $2`
			;;
		\-)
			echo `expr $1 - $2`
			;;
		\*)
			echo `expr $1 \* $2`
			;;
		\/)
			echo `expr $1 \/ $2`
			;;
	esac
}
calc $1 $2 $3
```

## 程序的返回值 {id="return-value"}

在 Shell 中，程序返回值有两种形式，分别是使用 `return` 命令以及使用 `echo` 命令。

当使用 `return` 命令的时候，需要注意一下两点事项:

- 只能返回 1 到 255 的整数
- 函数使用 return 返回值，通常只是用来提供其他地方调用获取状态，因此通常仅返回 0 或 1；0 表示成功，1 表示失败。

使用 `echo` 可以返回任意字符串结果，通常用于返回数据，比如一个字符串值或者列表值。

比如我们写一个检查 nginx 是否处于运行的函数:
```bash
this_pid =$$   ## 获取自身进程的 id
function is_nginx_running {
	ps -ef | grep nginx | grep -v grep | grep -v $this_pid &> /dev/null
  if [ $? -eq 0 ];then
  	return 0
  else
  	return 1
  fi
}
## 调用函数
is_nginx_running && echo "Nginx is running" || echo "Nginx is stoped"
```
再来举一个获取系统用户的例子:
```bash
function get_users {
	users=`cat /etc/passwd | cut -d: -f1`
  echo users
}
users=`get_users`   ## 获取函数调用的输出并赋值给 users 变量
fore user in $users
do
	echo $user
done
```

## 函数库 {id="libs'}

函数库的存在，是为了将经常使用的重复的代码，与具体业务无关的代码封装成文件。在下次用的时候，直接引用该文件即可。

首先，我们需要创建库文件，命名为 `base.lib`，并实现一个打印日期的函数:

```bash
#!/bin/echo
function print_date() {
	echo `date +百分号Y-百分号m-%d`
}
```
如何在其他的脚本文件中引入这个文件呢？使用：
```bash
#!/bin/bash

. ./base.lib  ## 引入库文件
print_date ## 调用库文件中的函数
```
在编写函数库的时候，需要注意以下事项：

- 库文件名的后缀是任意的，但一般使用 `.lib`
- 库文件通常是没有可执行权限的
- 库文件无需和脚本文件在同一个目录，只需要在脚本中引用时指定
- 第一行一般使用 `#!/bin/echo` , 输出警告信息，避免用户执行

## 总结 {id="summary"}

本文介绍了在Shell脚本编程中如何定义、使用函数以及处理函数返回值。首先，展示了两种定义函数的语法：一种是直接用圆括号包围函数体，另一种是通过
`function`关键字。函数传参时，参数通过`$n`引用。文中举例说明了如何编写一个简单的帮助信息输出函数和一个实现基础计算功能的计算器函数。

对于函数返回值，指出Shell中有两种方式：一是使用`return`命令返回1到255之间的整数值，通常用于表示成功或失败状态；二是使用`echo`返回任意字符串
结果，适合返回数据内容。通过实例阐述了一个检查Nginx服务运行状态的函数，并演示了如何根据返回值进行进一步操作。

另外，文章讨论了函数库的概念，即创建一个包含常用函数的文件（如`base.lib`），并通过`.`命令引入到其他脚本中调用函数。在创建函数库时，需要注意库
文件命名、权限设置、引用路径以及避免直接执行库文件等方面的细节。