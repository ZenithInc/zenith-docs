# 什么是 Linux

简单的说， Linux 和 Windows 、macOS 一样，都是操作系统，而操作系统本质上来讲也是一种软件(超大型的软件，代码量都在千万级以上)。

Linux 对于我们最大的意义在于，绝大多数的服务器上都使用 Linux 作为操作系统。我们的产品的开发、测试也都在 Linux 系统上进行操作。

## 搭建 Linux 环境 {id="started"}

搭建 Linux 环境大致有两种做法，一种是在本地(虚拟机)模拟一台服务器，然后安装 Linux 系统。另外一种就是选择云服务器厂商购买一台在线的服务器。

我更倾向于第二种方式，更加的快捷方便，但是需要一定的学习成本。第一种方式，对本地电脑的配置有一定要求，但是成本相对较低。

如果选择云服务器产品的话，可以选择阿里云、腾讯云、百度云、京东云、滴滴云、青云等等。滴滴云的成本最低，阿里云的应用较为广泛。对于初期学习而言，其实也都差不多，生产环境一般会选择阿里云或者腾讯云，如果是服务国外，可能会选择阿里云、Google Cloud Platform 以及亚马逊云。

## 什么是发行版 {id="release-version"}

在购买服务器的时候，会要求选择操作系统，当然我们会选择 Linux，另外还会要求选择 Linux 的发行版。那么什么是发行版呢？

简单的来说，我们平时所说的 Linux 指的是 Linux 内核(可以理解成 Linux 的核心）。不同的厂商、组织、开发者会针对某一版本的 Linux 内核做一些优化、加装一些软件形成自己的发行版本。

常见的发行版有 CentOS、RedHat、Ubuntu、Debian 等等。生产环境一般选用 CentOS, 而学习的话可以选择 CentOS 或者 Ubuntu。不严谨的定义，如下:

<note>
发行版 = Linux 内核 + 其他软件
</note>

接下来的系列文章，都会采用 CentOS 7.6 版本或者更高，推荐可以使用 Rocky Linux。

## 什么是命令行 {id="what-is-cli"}

相对于 Windows，我们所有的操作都是在图形化的界面中完成的，Linux 的所有操作都可以在命令行中完成。这样的操作更加的原始，但却更加的快速有效。

另外，Linux 的服务器一般都在异地的大型机房中，我们都需要通过网络连接去操作它。使用命令行传输指令相对于传出图形界面，会占用更小的带宽，传输更快。

Linux 的命令行界面一般如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/L7n4qbAYwTwhtledP9Rm.png" alt="cli" />

初学者对于这样的界面是困惑的，黑漆漆的背景，几个不知道含义的文本，不知道怎么做。但是我相信，你很快就会喜欢上这样的界面，因为它非常的高效，极客。

这就是命令行，你可以用各种各样的命令去做任何你想做的事情，前提是你需要理解命令的运作机制，这是学习 Linux 的第一道门槛，跨过去！

我们使用操作系统，通常是为了使用一些基于操作系统的软件，不管是聊天软件还是游戏软件或者是浏览器。因为在 Linux 中，我们通常都是使用命令行的，所以没有平时使用 Windows 中看到的哪些界面。而所谓的命令，其实就是一个个的软件，我们通过下面这种方式去运行，而不是通过鼠标去界面上点击：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/B3fhj6NZODL5IoDd8H5p.png" alt="qq"/>

你看，你平时使用微信或者QQ，是不是要鼠标双击 QQ或者微信的图标啊，那么在命令行中，我们并不需要用鼠标，只需要输入 QQ 就可以了。另外，当你打开QQ 的界面的时候，你需要告诉QQ你的用户名和密码，而在 命令行中使用过命令行参数来告诉运行的软件一些信息的, 比如 --username  和 --password 。

所以，你其实不用恐惧命令行，学习 Linux 其实就是学习这些软件怎么使用，就和你学习微信、QQ 如何使用一样，只是稍微比微信、QQ 这样的软件要复杂一点点。

所以，**究其本质来说，命令行和图形化界面没有什么区别，只是一个是通过文本来运行软件，一个是通过界面来简化软件的使用罢了** 。理解这一点很重要！

## 常用的命令

下面介绍一些常用的软件(命令), 不要去死记硬背，只要常常去用就好了。首先，我们创建一个文件夹, 这个文件夹名字叫做 `test` ：

```Java
## 创建一个名为 test 的文件夹
mkdir test
mkdir test test1 test2 test		## 创建多个目录
mkdir -p test/test/test/test	## 创建多级目录
mkdir -p test/{test1, test2, test3}   ## 创建父目录下多个并列的子目录
## 把当前的目录切换到名为 test 的文件夹
cd test
## 创建一个文件
touch test.txt
## 查看文本内容
cat test.txt
## 查看当前所在的目录
pwd
## 计算 1 + 1
expr 1 + 1
```

你会发现，如果熟练掌握了这些命令，那种用鼠标去点击各种按钮的操作会显得非常没有效率。接下来，我们来安装一个软件:

```Java
sudo yum install wget -y
```

相对上面的命令，这个看上去要复杂很多,解释一下:

如果你登录的账户是 root 用户，并不需要使用 `sudo` 就可以拥有安装软件的权限。因为 root 是 Linux 中权力做大的用户。否则，你可能需要使用 sudo 去提升你的权限，`sudo` 的意思就是 super to do ，使用超级管理员的权限去做某件事。

`yum` 是 CentOS 发行版中使用的软件管理工具，你可以使用它来安装一些软件，就像是 Windows 上的 360 软件管家。

`install` 是 `yum` 的子命令，表示安装。

<note>
不同的发行版中使用了不同的软件管理工具，在 RedHat/CentOS/Rocky Linux 中使用的是 `yum` 或者 `dnf`，在例如 Ubuntu 等发型版中使用的是 `apt`。
</note>

wget 是要安装的名字。而 `-y` 是说直接安装，不需要我确认了。

我举一个 qq 发送消息的例子:

```Shell
qq send hello -to bob
```

这条命令翻译过来就是: 使用 qq 这个软件发送给 bob 这个人一条消息，消息的内容为 "hi" 。

这个 wget 是干什么的呢？就像是 Windows 上的迅雷，用来下载文件的:
```Shell
wget http://file-linker.oss-cn-hangzhou.aliyuncs.com/B3fhj6NZODL5IoDd8H5p.png
```

用法非常简单，后面跟上文件的地址就可以了。

## 使用网络学习命令 {id="learn_more"}

其实命令行就这么简单，不需要死记硬背，用到的时候去查一下，多用用就好了。下面推荐一个地址 [LinuxCool](https://www.linuxcool.com/)，用来学习、搜索各种命令: 

这个网站对命令提供了搜索功能，对一些常见的命令还进行了分类。我更推荐向 GhatGPT 提出你的疑问。

## 使用 man 手册 {id="man"}

如果有一个人问我，这条命令怎么用，我会告诉他去问那个男人(man)。在 Unix/Linux 这个男人大名鼎鼎，比谁都更了解 Unix/Linux。其实 man 并不是指男人的意思，而是英文单词 manuals 的缩写，意为手册。

```Shell
man man
```
man 是 Unix/Linux 系统手册，我们可以使用 man 来查看 man 命令的帮助:

> man - an interface to the on-line reference manuals

它是一个在线的参考手册的接口，首先是参考手册，然后你自己的程序也可以实现它的接口，提供程序的帮助信息。
这个参考手册分为若干的章节(section)，如下:

| 章节   | 内容    |
|------|-------|
| 章节 1 | 一般的命令 |
| 章节 2 | 系统调用  |
| 章节 3 | C 库函数 |
| 章节 4 | 特殊文件  |

比如我们查询 sh 这条 Shell 的命令，可以使用如下方式:

```Shell
man 1 sh
```

然后关于 `sh`  这条命令的显示信息如下:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/jbAfwYVd0rotT6fgKx7r.png" alt="man sh"/>

## 更换镜像源 {id="image_source"}

由于某种不能抗拒的原因，国内访问国外的网站、服务都特别的慢。当我们使用 Yum 去安装各种软件的时候，由于服务器都在国外，经常会安装失败或者速度慢到不能忍受。

这时候，我们可以尝试更换成国内的镜像源。比如说阿里云、腾讯云等服务商会定时去同步国外 Yum 的官方服务器资源到国内自己的服务器上。当我们使用 Yum 去更新或安装的时候，就会比较快。

```Shell
## 备份原有配置文件
sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
## 下载源配置文件
sudo wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo
## 清理缓存并重建立
sudo yum clean all
sudo yum makecache
```
上面是 CentOS7 的实例，其他的发行版或不同的版本请访问[腾讯 CentOS 源帮助文档](https://mirrors.cloud.tencent.com/help/centos.html)。
