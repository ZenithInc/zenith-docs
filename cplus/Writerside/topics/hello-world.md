# 第一个 C++ 程序 

从这篇文档开始，我们正式学习 C++ 。当然，从最简单的 `Hello World` 开始。  

## Hello World {id="hello-world"}

首先我们来看看 Hello World 的代码在 C++ 中长什么样子? 

```C++
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

这段代码做了以下几件事情：

* `#include <iostream>` 这行代码告诉编译器包含标准输入输出流库，这是用来进行基本输入输出操作的。
* `int main()` 定义了主函数，这是每个 C++ 程序的入口点。
* `std::cout << "Hello, World!" << std::endl;` 这行代码使用 `std::cout` 输出字符串 "Hello, World!"，然后 `std::endl` 生成一个新行。
  `std::cout` 是 C++ 中用于控制台输出的对象，<< 是插入运算符，用于向输出流中插入数据。
* `return 0;` 表示程序正常结束。在主函数中，`return` 语句用于指定程序的退出状态。返回值 0 通常表示程序成功执行。

