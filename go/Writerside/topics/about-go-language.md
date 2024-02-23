# 关于 Go 语言

这篇文档对一些基础的内容进行总结整理，以便大家快速入门 Go 语言。

### Go 的历史 {id="history"}

Go 和其他语言以及其他技术都不是空中楼阁，都是站在其他语言的肩膀上产生的。我们看一下 Go 语言和其他语言的关系，下面这张图选择《Go 程序设计语言》一书:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/eBGubqqFyQdoFQqhSp9b.png" alt="history" />

> Go 总体上来说收到 C 语言的直接影响，属于 C 家族语言的分支。而声明与研发、包概念则受到了 Pascal 、Modula、Oberon 等语言的影响，二 CSP 则来自 Newsqueak 以及 Limbo 语言。


### Go 语言和其他语言的区别 {id="diff"}

下面这张表格对比了 Go 语言和其他几种主流编译语言的区别:

|            | 跨平台一次编码 | 一次编译 | 不需要运行环境 | 没有虚拟化损失 | 不需要自行处理GC | 面向对象 | 易用的并发能力 |
|------------|---------|------|---------|---------|-----------|------|---------|
| C          | ×       | √    | ×       | √       | ×         | ×    | ×       |
| C++        | ×       | √    | ×       | √       | ×         | √    | ×       |
| Java       | √       | ×    | √       | ×       | √         | √    | ×       |
| JavaScript | √       | ⭕    | √       | ×       | √         | √    | ×       |
| Go         | √       | ×    | √       | √       | √         | √    | √       |


### 采用 Go 语言编写的项目 {id="open_source_projects"}

下面这些项目都是采用 Go 语言编写的:

| Docker   | K8s   | etcd   |
|----------|-------|--------|
| codis    | BeeGo | falcon |
| bilibili | TIDB  | ...... |

### 环境变量设置 {id="env"}

在安装完成 Go 之后，需要设置两个环境变量:

```shell
# 启用模块，111 表示 1.11 版本
go env -w GO111MODULE=on
# 设置网络代理
go env -w GOPROXY=https://goproxy.cn,direct
```

### Hello World

我们遵循传统，从最基础的 `Hello World` 的示例开始讲解:

```go
package main		// 声明一个包，包名叫做 main

import "fmt"		// 引入一个库
// 声明入口函数，必须是 main
func main() {
    fmt.Println("Hello World!")   // 在终端打印 Hello World,调用了 fmt 的方法
}
```

> 你会发现，Go 语言的每一行代码后面不需要加上 `;` 符号。其实早起的 Go 程序是需要，在 2009 年 12 月后就取消了。但这并不意味着不需要封号，而是编译器帮我们补充了这个符号。


Go 的语法借鉴了 C、C++以及 Python。但是它对这些语言中的一些特性进行了发扬或者抛弃，比如说 C++ 一直被诟病太过于复杂，而 Go 语言只有 25 个关键字。在语法上，很多借鉴了 Python 的灵活。

运行这个程序，只需要在终端输入如下命令:

```go
go run hello.go
```

需要说明几点：

- 所有 Go 程序的代码起点，都是 mian 包中的 main 函数, main 包所在的文件是不是命名为 `main` 没有关系。

### 常量和变量

接下来，我们来说说在 Go 语言中的变量和常量，相对于其他的语言来说非常有自己的个性。

在 Go 语言中，有如下这些变量类型:

- `bool`, `string`
- `(u)int`, `(u)int8`, `(u)int16`, `(u)int32`, `(u)int64`, `unintptr`
- `byte`, `rune`
- `float32`, `float64`, `complex64`, `complex128`

Go 语言中并没有 `char` 这种类型，取而代之的是 `rune` 类型。`rune` 是 32 位的。

### Package

在 Go 语言中，使用 Package 来管理项目中的模块，类似于 Java 中的 Package 或者 PHP 中的命名空间。

声明一个 package,需要遵守下面这些规则:

- 一个目录下的同级文件必须使用同一个包名
- 包名可以与目录名称不同
- main 包是 Go 语言的入口包。一个 Go 语言程序必须有且只有一个 main 包。如果一个程序没有 `main` 包，编译报错，无法生成可执行文件。

我们可以使用 `import`  关键字导入其他包中的成员（常量、变量以及函数方法）。比如上面示例中的:

```go
import "fmt"
```

fmt 是单词`format`的缩写，用来格式化输入输出。

> 注意: 导入的包必须使用,否则编译不通过。


可以使用别名:

```go
import alias "fmt"
```

如果使用的别名是 `_`，那么就表示这个包不使用，则不能在代码中引用这个包。一般用于引入包做初始化。

### 变量的定义

在 Go 语言中，变量的定义需要指定类型，而类型的是写在变量名的后面的。为什么呢？因为 Go 语言觉得我们是先思考变量名，后思考变量类型的。定义如下:

```go
var name string = "Tom"		// 需要注意的是，Go 中的变量在定义之后就必须使用，否则编译不通过
```

如果不给予定义的变量初值，那么 Go 会使用默认值初始化变量，如下示例:

```go
var str string
var num int
fmt.Println(str, num)  // Output: '' 0
```

在一行代码中可以定义多个变量:

```go
var i, j int = 2, 4
```

Go 还支持类型推断，根据值来推断出变量的类型，如下示例:

```go
var a, b, c, d = 3, 3.12, "test", true
fmt.Println(a, b, c, d)
```

你还可以省略 `var` 关键字，使用一个 `:` 来替换它:

```go
a, b, c, d := 3, 3.12, "test", true		// 这种方式只能运用在函数内部，在函数外部必须使用`var` 关键字
```

> 注意: Go 语言并没有全局变量的说法，所有的变量要么是函数内的变量、要么是块变量、要么是包变量。


多个变量可以共享一个 `var` 关键字:

```go
var (
	a int
	b string
)
fmt.Println(a, b)
```

在 Go 语言中，有变量的类型是可以进行转换的。但是，这必须是强制类型转换，而不存在隐式类型转换。下面举一个例子:有一个直角三角形，其一边长为 4 cm，另一边长为 3 cm，请问其斜边长为多少？

```
a, b := 3, 4
result := int(math.Sqrt(float64(a * a + b * b)))
fmt.Printf("Result is %d", result)
```

### 常量的定义

常量可以使用 `const` 来声明:

```go
const Pi float64 = 3.1415926
const x, y int = 1, 3
```

我们也可以使用 `()` 来定义多个常量:

```go
const (
    White = 0xFFFFFF  // 十六进制表示法前面使用 0x
    Black = 0x000000
    Red = 0x00FF00
)
```

在 Go 语言中，常量定义如下:

```
const filename = "test.vim"
const a, b int = 3, 4
```

有一点和变量不同，常量的定义是可以不加以使用的。其他规则都和变量的定义类似，比如可以定义类型，也可以不定义类型。

还有一点和其他语言不一样，Go 语言中的常量习惯上并不以大写作为区分。

常量的定义也类似，如下:

```
const (
    java = iota
    php
    python
    goland
    typescript
)
fmt.Println(java, php, python, goland, typescript)
```

我们使用`iota`表示这一系列的枚举常量是递增自增长的。所以这输出的结果为`1 2 3 4 5`。

那么如果我要跳过`php`呢？可以使用如下这种方式定义:

```
const (
    java = iota
    _
    python
)
fmt.Println(java, python)
```

输出结果，java 的值为 0，python 的值为 2。

`iota`还可以是表达式，比如说我们要对存储单位进行转换(B、KB、MB、GB、TB、PB):

```
const (
    b = 1 << (10 * iota)
	kb
    mb
    gb
    tb
    pb
)

fmt.Println(b, bk, mb, gb, tb, pb)
```

输出结果如下:

```
1 1024 1048576 1073741824 1099511627776 1125899906842624
```

### 附录：Go 关键字

Go 语言的关键字非常少，只有 25 个，这也大大降低了 Go 语言的学习难度。对比 C++(20 标准), 有 87 个关键字之多，学习难度自然也提升了好多倍。下面罗列了 Go 语言中的所有关键字:

| continue | for         | import | return    | var    |
|----------|-------------|--------|-----------|--------|
| const    | fallthrough | if     | range     | type   |
| chan     | else        | goto   | package   | switch |
| case     | defer       | go     | map       | struct |
| break    | default     | func   | interface | select |

