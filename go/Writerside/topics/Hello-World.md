# Hello World

这篇文档将分析 Go 程序的编译过程，通过了解 Go 程序的编译更加深入的理解 Go 程序的生命周期。

首先我们来写一个 Go 最基础 Hello World 程序，文件命名为`hello.go`代码如下:

```go
package main

import "fmt"

func main() {
	fmt.Println("You are my sunshine!")
}
```

然后使用下面的命令打印这个程序的构建过程:

```shell
go build -n hello.go
```

输出的内容经过简化之后如下:

```Shell
# ...省略前面不重要的部分...
packagefile fmt=C:\Program Files\Go\pkg\windows_amd64\fmt.a
packagefile runtime=C:\Program Files\Go\pkg\windows_amd64\runtime.a
# ...中间省略...
"C:\\Program Files\\Go\\pkg\\tool\\windows_amd64\\compile.exe" -o "$WORK\\b001\\_pkg_.a"
packagefile command-line-arguments=$WORK\b001\_pkg_.a
packagefile fmt=C:\Program Files\Go\pkg\windows_amd64\fmt.a
packagefile runtime=C:\Program Files\Go\pkg\windows_amd64\runtime.a
packagefile errors=C:\Program Files\Go\pkg\windows_amd64\errors.a
packagefile internal/fmtsort=C:\Program Files\Go\pkg\windows_amd64\internal\fmtsort.a
# ...省略大部分依赖的 packagefile...
"C:\\Program Files\\Go\\pkg\\tool\\windows_amd64\\link.exe" -o # ...后面省略...
```

下面分析一下这个过程：

- 第 2 行和第 3 行，我们可以看到，我们的程序中包含了 `fmt.a` 和 `runtime.a` 文件，证明了程序会将 `runtime` 包作为程序的一同编译，一同运行
- 第  5 行：准备好编译的文件 `fmt.a`以及 `runtime.a` 之后调用编译器 `compile.exe` 进行编译
- 第 7 行到第 9 行： 输出程序依赖的所有的软件包，包括 `fmt` 、`runtime`、`errors` 以及更多省略的
- 第 12 行:  调用链接器 `link.exe` 将编译后的结果进行链接，产生最终的可执行文件

通过这个实验，我们可以看到 Go 语言的编译过程和 C/C++ 之类的语言是类似的，完整的编译过程如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/Qyf8oGVBt2NsRsgNK7O2.jpeg" alt="process"/>

**词法分析**是将程序转换为 Token（最小的语义结构，比如说 `package` 、`main` 、`"`这样一个个的单词）。

**句法分析**是在词法分析的基础上，将 Token 转为抽象语法树(AST)，类似如下这样的结构:

![](https://cdn.nlark.com/yuque/0/2022/jpeg/502915/1653595617652-36a213b7-6c78-4d84-a72b-086b810e1234.jpeg)

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/jlZcHhBFrVujTdpvf213.jpeg" alt="category"/>


**语义分析**是在抽象语法树的基础上，做如下这些事项的检查和优化:

- 类型检查:  因为 Go 是一门静态类型的语言，所以要在编译阶段进行类型的检查
- 类型推断：在大部分情况下，我们并不需要在代码中声明变量的类型，这是因为编译器在这时候会自动推断变量的类型
- 函数调用内联： 比如一些简单、短小的函数，编译器不会使用函数调用，而是直接使用函数体中的代码替换，减少函数的调用，以此来提升代码的性能
- 逃逸分析：分析变量到底应该存储在堆上还是栈上

**中间码(SSA)生成**这一步的目的是为了生成与平台无关的代码，就类似于 Java 中生成字节码，PHP 中生成 OPCode。主要是为了实现与平台无关的特性，不同的平台复用前面词法分析、句法分析、语义分析这些过程。当新增一种平台, 只需要实现中间码到特定平台的机器码这部分工作即可，不需要重复上面的工作。

下面这条指令可以查看`main` 函数生成的中间码:

```shell
$ export GOSSAFUNC=main && go build hello.go
# runtime
dumped SSA to C:\Users\happy\Workspace\coding.net\go\hello\ssa.html
# command-line-arguments
dumped SSA to .\ssa.html
```

你可以在浏览器中打开这条命令的输出结果中的 `ssa.html` 网页文件，可以看到到生成中间码为止上面每一步的输出，如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/kSYn3lcUVVEXqIepjXtw.png" alt="ssa.html"/>

最终生成的 SSA 代码如下:

```Shell
			# C:\Users\happy\Workspace\coding.net\go\hello\hello.go
			00000 (5) TEXT "".main(SB), ABIInternal
			00001 (5) FUNCDATA $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
			00002 (5) FUNCDATA $1, gclocals·f207267fbf96a0178e8758c6e3e0ce28(SB)
			00003 (5) FUNCDATA $2, "".main.stkobj(SB)
v35 	00004 (+6) MOVUPS X15, ""..autotmp_8-16(SP)
v14 	00005 (6) LEAQ type.string(SB), DX
v5 		00006 (6) MOVQ DX, ""..autotmp_8-16(SP)
v20 	00007 (6) LEAQ ""..stmp_0(SB), DX
v36 	00008 (6) MOVQ DX, ""..autotmp_8-8(SP)
v27 	00009 (?) NOP
			# $GOROOT\src\fmt\print.go
v30 	00010 (+274) MOVQ os.Stdout(SB), BX
v22 	00011 (274) LEAQ go.itab.*os.File,io.Writer(SB), AX
v13 	00012 (274) LEAQ ""..autotmp_8-16(SP), CX
v16 	00013 (274) MOVL $1, DI
v7 		00014 (274) MOVQ DI, SI
v32 	00015 (274) PCDATA $1, $0
v32 	00016 (274) CALL fmt.Fprintln(SB)
			# C:\Users\happy\Workspace\coding.net\go\hello\hello.go
b4 		00017 (7) RET
			00018 (?) END
```


**代码优化**，这一步其实是贯穿整个编译过程的，也就是说在编译中每一步都可以进行优化。

**机器码生成**，这一步会生成 Plan9 汇编代码，这个 Plan9 汇编代码是和具体的平台相关的， Linux 下生成的汇编和 Windows 下生成的汇编是不同的。然后将对应平台的 Plan9 汇编编译成机器码，机器码的文件以 `.a`结尾。

我们可以通过下面的命令来查看最终生成的 Plan9 汇编代码:

```shell
$ go build -gcflags -S hello.go
# runtime
dumped SSA to C:\Users\happy\Workspace\coding.net\go\hello\ssa.html
# command-line-arguments
dumped SSA to .\ssa.html
"".main STEXT size=103 args=0x0 locals=0x40 funcid=0x0 align=0x0
        0x0000 00000 (C:\Users\happy\Workspace\coding.net\go\hello\hello.go:5)  TEXT    "".main(SB), ABIInternal, $64-0
        0x0000 00000 (C:\Users\happy\Workspace\coding.net\go\hello\hello.go:5)  CMPQ    SP, 16(R14)
        0x0004 00004 (C:\Users\happy\Workspace\coding.net\go\hello\hello.go:5)  PCDATA  $0, $-2
        0x0004 00004 (C:\Users\happy\Workspace\coding.net\go\hello\hello.go:5)  JLS     92
        0x0006 00006 (C:\Users\happy\Workspace\coding.net\go\hello\hello.go:5)  PCDATA  $0, $-1
# ...下面的汇编太长，省略...
```

**链接**， 这一步就是把生成的所有的 `.a` 结尾的机器码文件进行链接，生成最终的可执行文件。

