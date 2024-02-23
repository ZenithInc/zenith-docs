# 搭建开发环境 

这篇文档将来介绍在各个操作系统下，开发环境的安装。只有在准备好了开发环境之后，才能更好更有效率地学习使用 C 语言。

## Linux 环境安装 {id="linux-env"}

在 Linux 下安装 C 的开发环境要简单很多，使用命令行安装 Gcc、CMake、Gdb、Make等软件即可，IDE 的话可以选择 CLion。

## Windows 下环境安装 {id="windows-env"}

首先下载一些软件，比如 MSVS、MSYS2、CLion。然后配置 MSVS, 选择 C++ 桌面开发场景，大概要下载两个多 GB 的数据，然后重启操作系统。

接着来配置 MSYS2 的镜像源，留下国内的镜像源，这样下载快一些，默认安装路径下修改 `C:\msys64\etc\pacman.d` 目录下的配置文件，一共有三个，全部修改掉:

```
Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/i686/
Server = http://mirrors.ustc.edu.cn/msys2/mingw/i686/
```

然后启动 MSYS2 来安装一些开发者工具:

```shell
pacman -Su		# 更新所有的软件
pacman -Sy base-devel  # 安装开发者工具
pacman -S mingw-w64-x86_64-toolchain # GCC 等编译器
```

## CLion 配置 {id="clion-configuration"}

配置 CLion开发工具链:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/UgvQvFtS2s8TryWKVt9U.png" alt="clion configuration"/>

> 在配置 MinGW 的时候需要注意， CLion 目前支持的 GDB 的版本最高为 9.2.x, 但是 MinGW 使用 pacman 更新之后都会采用最高版本(当前是10.x)。所以需要使用更低的版本，CLion 自带了 GDB 9.2 版本，默认在 `C:\Users\admin\AppData\Local\JetBrains\CLion 2020.2.4\bin\gdb\win\bin\gdb.exe` 路径下。


配置 DEBUG:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/eVv6BUx3pKU2MYWEd8gB.png" alt="configuration debug in clion"/>

## CLion 使用技巧 {id="how-to-use-clion"}

首先来写一个示例程序, 如下:
```c
#include <stdio.h>

int main() {
	for (int i = 0; i < 5; ++i) {
		printf("Hello World\n");
 	}
}
```
下面将针对这个示例程序来演示一些 CLion 的使用技巧。

### 设置 Google 的代码风格 {id="setting-google-code-style"}

为了代码风格的统一，写出更优雅的代码，我们需要选择一种代码风格，这里选择使用 Google 的代码风格，设置如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/xWghvfHvqs5MhOj1OHRJ.png" alt="code style setting"/>

但是有些选项默认并没有启用，比如说 Google 的代码风格要求所有的变量名采用小写加上下划线，如何启用呢?

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/1yXOIbbrerjaQbKcyRtB.png" alt="code style setting"/>

### Debug {id="configuration-debug"}

首先介绍如何 Debug 这个程序，比如打断点、单步调试、查看值、使用监视器、查看内存数据等。先来说打断点，比较简单，只需要在行号的右边单击一下就可以了, 会出现一个红色的圆点，如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/NhUraGLaXfr0hWNlCM9T.png" alt="debug"/>

然后使用 Debug 运行就会出现 Debugger 窗口, 如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/EBaGN0npRbpuTjECFePV.png" alt="debug"/>

然后使用快捷键 `ALT+F8` 可以在弹出的 Evaluate 窗口，输入表达式并查看对应的结果:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/OzMvQiNIUY39Xa2OBLrv.png" alt="debug"/>

在 LLDB 窗口使用 `disassemble`命令可以查看反编译的结果:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/8JAFEKVrL5Z2Rvm1mEIa.png" alt="debug"/>

在 Variables 窗口，选中 `i` 变量，使用快捷键 `CTRL+ENTER` 可以查看 `i` 变量在内存中的值。

调试的快捷键如下表所示:

| 快捷键          | 含义      |
|--------------|---------|
| F9           | 跳到下一个断点 |
| F7           | 步入      |
| ALT+SHIFT+F7 | 强制步入    |
| SHIFT+F8     | 步出      |
| F8           | 下一步     |


### Compiler Explorer 插件

借助汇编，也有助于对于 C 语言的深入理解。而使用这个插件可以查看对应代码的汇编指令，支持高亮显示，非常的方便。安装好插件之后需要进行如下设置:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/SQXgeTawnmGQOsberZZy.png" alt="compiler explorer"/>

我在 MacOS 上并没有成功的运行这个插件，也可以使用在线网站在线编译查看汇编结果：[Compiler Explorer](https://godbolt.org/)。

## 总结 {id="summary"}

这篇文档介绍了如何搭建 C 语言的开发环境，包括在 Windows 上以及 like-unix 的操作系统上。介绍了 CLion 的使用，以及如何在 CLion 上进行 Debug。

