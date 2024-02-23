# 退出状态码

在 Shell 脚本中，每一行命令在执行完成之后，都会在 Shell 执行环境中保存其退出的状态码。

* 所有的 Shell 命令都使用退出状态码来告知 Shell 它执行结束了。
* 退出状态码是一个 0 ～ 255 的整数值。
* Linux 提供了一个 $? 的特殊变量来获取上一条命令执行结束后的退出状态码的值。

如下示例，展示了状态码的使用:
```Shell
echo 'Hello' ## 输出: Hello
echo $?  ## 输出: 0, 表示上一条 echo 'Hello' 的命令执行成功
```

在 Linux 中提供了一些常用的退出状态码，如下表所示:

| 状态码   | 	含义                   |
|-------|-----------------------|
| 0     | 	命令执行成功并结束            |
| 1     | 	一般性未知错误              |
| 2     | 	不适合的 Shell 命令        |
| 126   | 	命令不可执行               |
| 127   | 	没找到命令                |
| 128   | 	无效的退出参数              |
| 128+x | 	与 Linux 信号 x 相关的严重错误 |
| 130   | 	通过 Ctrl + C 终止的命令    |
| 255   | 	正常范围之外的退出码           |

在 Shell 脚本编程中，我们没有必要记住所有的这些状态码。但需要注意以下几点:

* 0 表示命令执行成功的情况
* 非 0 值表示命令执行不成功的情况

我们来看一下执行错误的示例:
```Shell
fdjafldajfdla ## 输出: fdjafldajfdla: 为找到命令
echo $? ## 输出: 127 表示没有找到命令
```

然后，我们来看一个实际的例子，检查某项服务是否已经启动:
```Shell
#!/bin/bash

ps -ef | grep $1 | grep -v grep | grep -v sh
if [ $? -eq 0 ];then
	echo 'already start...'
else
  echo 'failed...'
fi
```

来执行这个脚本，如下:
```Shell
sh service.sh nginx   ## 如果没有启动 Nginx，输出: failed...
sh service.sh systemd ## 如果启动了 systemd，输出: already start...
```

上面的示例中，状态码的具体的值都是系统默认定义的。此外，我们也可以自定义状态码的输出，使用 exit 命令：
```Shell
## 使用内联输入重定向创建测试的脚本文件
cat <<EOF > exit.sh
#!/bin/bash
exit 12
EOF
sh exit.sh  ## 执行脚本文件
echo $? ## 输出: 12
```