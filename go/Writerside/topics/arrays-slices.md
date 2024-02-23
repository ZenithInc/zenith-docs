# 数组和切片 

Go语言中，数组和切片是常用的数据结构，用于存储一系列相同类型的元素。数组是一种固定长度的数据结构，而切片则是对数组的一个可变长度的引用。它们在Go语言中都有着重要的作用，并且在不同的场景中有不同的用途和特点。

## 数组 {id="arrays"}

在 Go 语言中，数组是固定的一组相同类型的数据，可以使用从 0 开始的下标访问其对应的元素。这一篇文档我们聊介绍这一个数据结构——数组。

### 数组的声明和初始化

声明一个数组的时候，必须要给与确定的长度:

```go
var arr = [5]string{"PHP", "Java", "Go", "C++", "C"}
// 或者
arr := [5]string{"PHP", "Java", "Go", "C++", "C"}
// 或者
arr := [...]string{"PHP", "Java", "Go", "C++", "C"}
```

我们也可以使用初始化个别的值:

```go
arr := [5]string{4:"PHP"}   // 将第四个元素初始化为 PHP
```

需要注意的是，在 Go 语言中，数组是值类型的，而不是引用类型的。这一点和 C 语言不一样，在 C 语言中，数组和字符串都是引用类型的。在 Go 语言中，字符串底层也是字节数组，但是不可变的。

数组的长度可以使用 `len` 函数:

```go
arr := [...]int{}	// 空数组
fmt.Printf("length is %d\n", len(arr))	// 长度为 0
```

在控制台打印数组如下:

### 数组的遍历

如何对数组进行遍历呢？如下示例:

```go
var arr = [5]string{"PHP", "Java", "Python", "Go", "C++"}
for i := 0; i < len(arr); i++ {
	fmt.Println(arr[i])
}
```

上面的示例中我们使用了 `len` 这个 Go 内置的函数获取数组的长度，使用了 `for` 来遍历它。但是，使用 `for` 和 `range` 更加的方便，类似于其他语言中的 `foreach` ,并不需要知道数组的长度:

```go
arr := [5]string{"PHP", "Java", "Python", "Go", "C++"}
for index, value := range arr {
    fmt.Println(index, value)
}
```

大概每一种语言都会有自己非常依赖的一种数据结构，比如说 PHP 依赖数组(Array), Python 依赖列表(List), 而 Go 语言依赖切片(Slice)。所以，掌握切片对于用好 Go 来说是非常重要的。这篇文档就来描述切片的相关内容。

## 切片

切片是一种动态的数组，其底层结构也是使用了数组。

### Slice 的多种定义方式 {id="defined_slice"}

切片的定义方式和数组也非常相似，如下示例:

```go
// 定义一个数组，数组需要指定长度
arr := [5]string{"PHP", "Python", "Go", "C", "C++"}
fmt.Printf("%T\n", arr)			// Output: [5]string
// 定义一个切片，切片是动态的数组，所以并不需要指定长度
slice := []string{"PHP", "Python", "Go", "C", "C++"}
fmt.Printf("%T\n", slice)		// Output: []string
```

除此之外，我们还可以使用 `make` 来初始化一个 Slice：

```go
slice := make([]string, 6)
fmt.Println("%d\n", len(slice))  // Output: 6
```

`make` 需要传递两个参数，第一个是 slice 的类型，第二个是 slice 的长度。

然后我们再来介绍第三种方法，使用数组来初始化一个 Slice:

```go
arr := [5]string{"PHP", "Python", "Go", "C", "C++"}
slice = arr[0:4]
fmt.Printf("Type %T, Value is %s\n", slice, slice[3])
```

需要注意的是， `arr[0:4]` 从 `arr` 中获取一个区间，这个区间是 **左闭右开，**即从第 0 个到第 3 个，并不包括第 4 个。如果要获取整个数组，可以使用 `arr[:]` 。

**Slice 是引用传递，而 Array 是值传递。所以，从 Array 初始化 Slice，如果改变了 Slice 也会相应的改变原来的 Array。**如下示例:

```go
arr := [5]string{"PHP", "Python", "Go", "C", "C++"}
slice = arr[0:4]
slice[1] = "Java"
fmt.Println(arr[1]) // Output: Java
```

然后我们再来，定义第四种方式，使用 `new` 关键字:

```go
slice = new([]string)
```

### Slice 的基本操作

因为 slice 是一个动态的数组，我们就可以对它动态的添加、修改、删除值。下面的例子，我们演示了如何追加一个值:

```go
slice := []string{"Go", "PHP", "Java", "C++"}
append(slice, "C")	// 追加一个值
append(slice, "VB", "C#")   // 追加多个值
```

我们也可以通过 `copy` 函数来追加其他切片中的值:

```go
sourceSlice = []string{"Go", "PHP", "Java", "C++"}
targetSlice := make([]string, len(sourceSlice))
copy(sourceSlice, targetSlice)
fmt.Printf("%v\n", targetSlice)
```

从切片中删除元素:

```go
slice := []string{"Go", "PHP", "Java", "C++"}
slice2 = append(slice[:1], slice[2:])	// 使用 append 来合并
fmt.Printf("%v\n", slice2)
```

### 基于数组的结构体

Slice 的本质就是基于数组的结构体，这个结构体定义在 `runtime/slice.go` 文件中，如下:

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

通过这个 `slice` 的结构体，我们可以知道切片实际上是对数组的引用。那么 `len` 和 `cap` 这两个数值指的是什么呢？我们可以通过下图来了解:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/vvn99A2LMZtWfOVN610E.jpeg" alt="img1"/>

根据上图，我们可以看到`len` 表示的是数组元素已经使用的长度，而 `cap` 表示的是数组总的长度，而 `cap` 减去 `len` 得到的是数组剩余的容量。

接着我们可以通过输出汇编指令来看 slice 的创建, 创建 Slice 的代码如下：

```go
package main

import "fmt"

func main() {
	s := []int{4, 5, 6}
	fmt.Println(s)
}
```

编译并输出汇编指令:

```shell
$ go build -gcflags -S main.go
...省略前面的输出...
0x0018 00024 (main.go:6)        LEAQ    type.[3]int(SB), AX
0x001f 00031 (main.go:6)        PCDATA  $1, $0
0x001f 00031 (main.go:6)        NOP
0x0020 00032 (main.go:6)        CALL    runtime.newobject(SB)
0x0025 00037 (main.go:6)        MOVQ    $1, (AX)
0x002c 00044 (main.go:6)        MOVQ    $2, 8(AX)
0x0034 00052 (main.go:6)        MOVQ    $3, 16(AX)
...省略后面的输出...
```

我们看到第六行生成的汇编指令，解释如下：

- 第 3 行：创建了一个大小为 3 的数组
- 第 6 行： `runtime.newobject(SB)` 创建了一个新结构体的值
- 第 7 行到第 9 行：将三个变量存入 Slice 的结构体

上面对应的代码在 `runtime/slice.go` 中的 `makeslice` 方法。

### 切片的扩容

当我们使用 `append` 函数对切片追加元素，如果原有的数组结构长度不足以容纳，将会隐式地触发扩容机制，使用扩容算法产生新的数组。其扩容算法如下:

如果新申请容量(Cap)大于2倍地旧容量(Old Cap)，那么新生成地数组地容量就为就是申请容量(Cap)。如果不是，接着判断旧 Slice 地长度小于 1024，则最终地容量就是旧容量(Old Cap) 的两倍，即(new cap=doublecap)。否则，就接着判断，旧切片的长度大于等于 1024，则最终容量(New Cap)从旧容量(Old Cap)开始循环增加原来的 1/4，即(New Cap = Old Cap, for {New Cap += NewCap / 4})直到最终容量大于新申请的容量(Cap)。如果最终容量(cap)计算值溢出，则最终容量就会是申请容量 。

需要注意两点：

- **如果触发了扩容机制，指针就会指向新创建的数组的地址** 。
- **切片的扩容操作，是并发不安全的，需要加锁处理**。
