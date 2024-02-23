# tmux

使用 Linux，我们绝大多数的时候看到的都是黑漆漆的画面，是有一些枯燥的。但是，能不能在枯燥的画面中，玩些花样呢？

比如说，我们模拟一个多窗口的环境，可以再多个窗口之间切换。再比如说，模拟一个多桌面的环境，不同的桌面包含不同的窗口。

要实现上面这些功能，我们就需要清楚这篇文章的主角了 —— tmnx。

### 安装 tmux {id="install"}

如果是 CentOS：
```shell
$ sudo dnf install tmux # CenOS7 使用 yum
```
如果是 macOS:
```shell
$ brew install tmux
```
### 创建和销毁会话 {id="create-and-destroy-session"}

在 tmux 中，有一个 Session 的概念，和桌面是一样的意思。一个 Session 可以包含多个窗口( Window ）。如何创建 Session 呢？

```shell
$ tmux new -s <Session Name>
```

查看已经创建的 Session：
```shell
$ tmux ls
```
进入已经创建的 Session：
```shell
$ tmux a -t <Session Name>
```
如果要临时退出会话, 使用 `ctrl+b`，然后按下 `d`。

如果要销毁会话，使用 `ctrl+b` ，然后按下 `:` 进入命令模式，输入命令: `kill-session` 。

## 模拟多窗口的环境 {id="window"}

窗口是基于 Session 的，所以需要先进入一个 Session。然后创建窗口，需要使用 `Ctrl + b` , 然后按下  `c` 键:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/qbekEIQaMeuTD9O9Js78.png" alt="create session"/>

我们看到下面显示着一行绿色的状态栏，内容如下:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/D54Pvg6IxtiToOkrpfBb.png" alt="status bar tmux"/>

你看到上面切换的窗口序号了吗？我们可以根据这个切换窗口，比如要切换到第一个窗口，就可以按下组合键: `Ctrl + b` , 然后按下 `1` ，这里的 1 就是窗口的序号。

我们看到，起 Window 的默认命令规则是采用了命令提示符，如果要重命名呢？ `Ctrl+B` ,然后按下 `,` 。

如果我们要暂停这个会话，回到最初的界面呢？ `Ctrl + B` ，然后按下 `d` 。

## 面板(Plane)的相关操作 {id="plane"}

下表展示了一些常用的场景以及对应的面板的操作：

| 场景            | 操作                                      |
|---------------|-----------------------------------------|
| 面板切换          | `CTRL + B`, `CTRL + o`，注意， `o` 要快速按下并释放 |
| 面板排序          | `CTRL + B`, `CTRL + o`, 注意，`o` 要长按一小会   |
| 面板全屏          | `CTRL + B` ,`CTRL + z`                  |
| 面板调整大小        | `CTRL + B`, `ALT + <方向键>`               |
| 创建水平面板        | `CTRL + B`，然后按下 `"`                     |
| 创建垂直面板        | `CTRL + B`，然后按下 `%`                     |
| 水平面板和垂直面板相互切换 | `CTRL + B`, 然后按下空格键                     |

这里要注意面板的切换和面板的排序，按键都是一样的。区别时按下 `o`键时候的时间长短，经常会因为动作的延迟将面板切换操作成面板排序......

## 外观美化 {id="status-bar"}

美化的代码如下:

```shell
$ git clone https://github.com/gpakosz/.tmux.git
$ ln -s -f .tmux/.tmux.conf
$ cp .tmux/.tmux.conf ~
```
效果如下图:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/BdCLklWXoXJ77zDKcyaL.png" alt="status bar tmux"/>
