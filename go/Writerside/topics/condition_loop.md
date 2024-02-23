# &#26465;&#20214;&#21644;&#24490;&#29615;

这篇文档描述了如何在 Go 程序中使用具有 Go 特色饿分支和循环语句。

## 分支语句

接下来，介绍条件语句。我觉得一个好的程序应该减少条件语句，但是不能没有条件语句。就像我希望生活能少一点变化，但是不能没有变化一样。

在 Go 语言中，条件语句中的条件是不需要用括号包裹的，这是和其他语言比较大的一个区别，可能是因为 Go 追求的是一种简洁，所以少一些关键字也少一些符号吧。

```go
score := 10
if score == 10 {
    // todo
} else if score > 10 {
    // todo
} else {
    // todo
}
```

上面这个程序没有任何意义，只是为了说明 `if...else...`语句在 Go 语言中的表现。再来一个稍微有些实际意义的段子，读取一个文件，当这个文件存在则输出文件内容，否则输出错误信息：

```go
const filename = "test.txt"
contents, err := ioutil.ReadFile(filename)
if err != nil {
    fmt.Println(err)
} else {
    fmt.Println(string(contents))
}
```

我们也可以将 `if` 的条件部分改写成如下形式，代码看上去会更简洁:

```go
const filename = "test.txt"
if contents, err := ioutil.ReadFile(filename); err != nil {
    fmt.Println(err)
}
```

接下来在看看 switch 语句在 Go 语言中的呈现:

```go
func eval(a, b int, op string) int {
	var result int
	switch op {
	case "+":
		result = a + b
	case "-":
		result = a - b
	case "*":
		result = a * b
	case "/":
		result = a / b
	default:
		panic("unsupported operator:" + op)
	}
	
	return result
}
```

Go 语言和其他语言不一样的是，不需要在 case 末尾显式地写上 break，默认就是有的。这样一来，妈妈再也不用担心我忘了写 break 了。如果一定要不用 break，那么就需要写成如下形式:

```go
case "*":
    result = a * b
	fallthrough
```

switch 语句也可以没有表达式，这种情况下需要在 case 中写明条件:

```go
switch {
    case score < 90:
        // todo
    case score < 80:
        // todo
}
```

此时外，还支持下面这些特性:

- 一个分支多个值，例如 `case "golang", "java":` ；
- 分支后面可以是常量，也可以是表达式，如上面的示例中那般；

## 循环语句

在 Go 语言中，循环非常有特点。我们用三个例子来说明:

```go
for {
    fmt.Println("Hello")
}
```

这就是死循环，在 Go 语言中，经常要使用死循环来并发编程，所以将死循环设计地如此简洁。再来看第二个例子,十进制数转成二进制：

```go
func convertToBin(v int) string {
	result := ""
	for ; v > 0; v /= 2 {
		result = strconv.Itoa(v%2) + result
	}
	return result
}

func main() {
	fmt.Println(convertToBin(2),
		convertToBin(10),
		convertToBin(1024))
}
```

第三个例子，我们改写前文中读取文件的程序。之前是一次性读取文件所有内容，现在逐行读取文件内容:

```go
func main() {
	filename := "test.txt"
	file, err := os.Open(filename)
	if err != nil {
		panic(err)
	}
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
}
```

在这个例子中，我们使用 Scanner 来逐行读写文件。在 Go 语言中，并没有 foreach 或者是 while 之类的，使用 for 通过省略初始条件和步长就可以达到相同的效果。

在来看一个例子，使用 `for` 来遍历切片：

```go
func main() {
	for key, value := range []int{0, 1, 2, 3, 4, 5} {
		fmt.Println("Key: %i / Value: %i", key, value)
	}
}
```

遍历 `map` 也是一样的:

```go
func main() {
	for key, value := range map[string]int {
		"go": 1,
		"php": 2,
		"java": 3,
	} {
		fmt.Println("Key: %i / Value: %i", key, value)
	}
}
```

## 编译器对分支和循环的性能优化

在Go语言中，编译器对循环和分支语句进行了一些优化，以提高代码的执行效率。以下是一些常见的优化技术：

* 循环展开（Loop Unrolling）：编译器会尝试将循环展开，即将循环体内的代码复制多次，减少循环迭代的次数。这样可以减少循环控制的开销，提高代码的执行速度。

* 分支预测（Branch Prediction）：编译器会分析条件语句的分支情况，并根据统计信息预测分支的走向。这样可以使得CPU的分支预测器更准确地预测分支，减少分支跳转带来的性能损失。

* 循环不变量提取（Loop Invariant Code Motion）：编译器会将循环内部不依赖于循环变量的计算提到循环外部，避免重复计算。这样可以减少循环内部的计算量，提高代码的执行效率。

* 编译器优化指令重排（Compiler Optimization Instruction Reordering）：编译器会根据指令的依赖关系进行重排，以提高指令的并行度和执行效率。这样可以充分利用CPU的多级缓存和乱序执行等特性，提高代码的执行速度。

需要注意的是，编译器的优化并不是绝对的，可能会因为代码的结构、复杂性等因素而有所差异。此外，编译器优化也可能会导致代码的行为发生变化，因此在进行性能优化时，需要进行充分的测试和验证。

## 总结

总的来说，Go语言中的分支和循环语句相对简洁而灵活，没有过多的语法糖，使得代码更易读、易懂。同时，Go语言的编译器会对循环进行优化，提高执行效率。