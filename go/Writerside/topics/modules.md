# 模块 

在编程中，有一个问题是贯穿始终的，那就是**代码应该放在哪里?小的函数的封装，再到代码的目录结构，上升到代码的设计原则，最终到分层架构，都是围绕着这个问题展开的。**

Go 语言在设计之初对于代码放在哪里并没有一个长久的规划，设计了 GoPath 的做法，但这为人所诟病，不是那么好用。再到后来才推出了模块(v1.12之后)的概念，才有了一个相对合理的解决方案。

写代码的人应该记住，代码放在哪里是一个时时刻刻都应该思考的问题。这一篇文档，我们就来说说，在 Go 语言中，我们的代码应该如何存放。

### 模块化(Go Modules)

Go 语言在 v1.12 版本之后，推出了 Go Modules 这种形式来管理我们的代码包。我们可以使用如下命令来初始化一个模块:

```shell
go mod init <模块名>
```

执行这条命令之后，就会在当前目录下创建一个名为 `go.mod` 的文件。在使用模块的时候，要注意以下几点:

- 这个目录下创建任何的目录自动成为这个模块的子模块。
- 每个代码文件中，都必须使用 `package` 声明包名，相同文件夹下的 `package` 都必须一致。
- 代码的运行都是从 `main` 包的 `main` 函数开始的。

我们来创建一个做一个最小化的案例试试看:

```shell
mkdir example && cd example		# 示例代码都存放在这个目录下
go mod init example						# 初始化模块
mkdir subpack									# 创建一个子模块的目录
```

然后创建文件 `subpack/product.go` ，内容如下:

```go
package subpack
// 创建一个“产品”的结构体，在其他包当中引入
type Product struct {
    Name string
}
```

编写名为 `entry.go` 的入口文件内容如下:

```go
package main

import "fmt"
import "example/subpack"	// 引入的是包的路径，而不是包名

func main() {
    var p = subpack.Product{
        Name: "iPhone Pro Max 128G Green",
    }
    fmt.Println(p) // Output: {iPhone Pro Max 128G Green}
}
```

需要注意的是 `example/subpack` 是包的路径，而不是包名。当我们的包所在的文件夹名字和包名一致的情况下是一样的，实际上我们的包名和文件夹名字是可以不一样的。

### 使用 go mod 来管理依赖

在项目中，我们经常会说“拿来主义”，意思就是如果别人已经写好的代码，在版权允许的情况下，拿来即用。如何拿？拿来如何管理呢？

Go Module 解决了两个问题，一个是大家写的代码都按照固定的规则存放在磁盘上，我们也可以按照这套固定的规则来引入代码。换句话说，我们可以随意集成别人的代码，按照 Go Module 的方式引入别人的代码。另一个解决的问题是，Go  提供了命令行工具 `go get` 来轻松管理项目里引入的包的依赖关系。

我们以 Uber 开源的高性能日志库为例子，来看看 Go 是如何管理依赖的。首先，我们先通过 `go get` 来获取这个库:

```shell
$ go get -u go.uber.org/zap # -u 参数的意思是，如果已经存在该包，则更新，如果不加这个参数则会跳过这个包
# 你也可以指定版本，比如
$ go get -u go.uber.org/zap@v1.16.0
```

完成之后，我们的目录结构发生了变化。第一个变化是 `go.mod` 文件中多了如下内容:

```
require (
	go.uber.org/multierr v1.6.0 // indirect
	go.uber.org/zap v1.16.0 // indirect
)
```

意思是我们的项目中包含了这些依赖包以及对应的版本，其中 `multierr` 这个依赖包是哪里来的呢？这个是 `zap` 这个包依赖的。所以我们的软件的依赖最终会形成一颗树。我们不用管，Go 会替我们将所有的依赖管理理清楚并管理好。

另外一个变化是，多出了一个文件 `go.sum` ，截取的内容如下:

```shell
go.uber.org/zap v1.16.0/go.mod h1:MA8QOfq0BHJwdXa996Y4dYkAqRKB8/1K1QMMZVaNZjQ=
```

这个文件中记录着我们各种依赖的包的散列值，用来检测我们下载下来的包是不是正确的、完整的。你是还存在疑问？那么下载下来的依赖包的代码放在哪里呢？我觉得 Go 的设计故意不让你知道它放在哪里，因为你如果要看源码可以直接从 IDE 中点击响应的函数跳转过去，具体放在哪里不用关心。所以，你在项目的根目录下看不到依赖的项目：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/ROFCTLtfs5lK86mIuW9j.png" alt="img1"/>

如果我希望项目中使用的这个依赖降级或者升级呢？只需要获取新的版本即可:

```shell
$ go get -u go.uber.org/zap@v1.12   # 当前版本是 1.16
$ go get -u go.uber.org/zap   # 升级到最新版本
go get: upgraded go.uber.org/zap v1.12.0 => v1.16.0
```

另外，当你在代码中引入了没有引入的依赖，在编译代码的时候会自动去拉取依赖的代码。如果项目中存在一些不用的依赖，需要清理，可以使用:

```shell
$ go mod tidy
```

当你不希望使用网络来管理依赖的时候，可以使用下面的命令将所有依赖的软件包缓存到本地:

```go
$ go mod vendor
```

如果你想将自己写的软件包发布，并提供给别人依赖的话。只需要执行下面的命令，生成一个 `go.mod` 文件，并将代码传到公开的 Git 仓库即可。别人通过 `go get` 命令就可以拉取。

```shell
go mod init <git url>
```

增加新的版本的时候，只需要使用 git 打上一个新的 tag 就可以了。

### import 详解

如果要导入多个包，我们可以使用括号来减少重复输入 `import` :

```go
import (
    "fmt"
    "example/subpack"
)
```

我们在使用 `fmt.Println` 函数的时候，有的人可能会觉的重复去写 `fmt` 比较烦，我们可以使用另一种引入方式:

```go
import . "fmt"
// 使用
Println("Hello World")
```

在使用包的时候，经常会遇到包名重复的情况，我们可以给包的路径取一个别名(Alias)：

```go
import alias "fmt"
// 使用
alias.Println("Hello World")
```

### 匿名引用

另外，如果我们引入了不使用的包，那么 Go 语言从语法层面就会报错，编译不会通过。如果仍旧希望引入暂时不使用的包，我们可以使用匿名引用:

```go
import _ "fmt"
```

这样做有什么意义吗？是有深意的。如果我们的包当中有一些 `init` 函数的时候，使用匿名引入的时候虽然没有使用这个包，但是会执行包当中一个或者多个的 `init` 函数(在多个文件中都有 `init` 函数的情况下，是无法保证其这些函数的调用顺序的)。

另外，使用 `import` 的时候还需要注意两点:

- 允许重复引入
- 不允许循环引入，比如 A 包引入了 B 包，B 包引入了 C 包，C包再次引入了 A 包，编译不会通过

<note>
再见，GoPath, GoVendor
</note>