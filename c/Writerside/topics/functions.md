# 函数 

函数并不是计算机中才有的概念，而是来源于数学中的概念。但是计算机中的函数和数学中的函数的目的是不一样的，数学中的函数是为了研究两个集合之间的关系的，而计算机中的函数的目的是为了代码逻辑的复用。这一篇文档我们来说说 C 语言中的函数的相关内容。

## 函数的语法 {id="syntax"}

在 C 语言中，函数的语法定义如下:
```c
<return value type> function name(<parameters>) {
 	statements
    return <value>
}
```
比如说一个求和函数的定义如下:
```c
int sum(int left, int right) {
 	return left + right;   
}
```
> 注意：C 语言中的函数的定义和其他的语言在传参和返回值上是存在不一样的。如果没有声明返回值，则表示可以接收任意的返回值。如果返回值没有定义，则默认表示接收 `int` 类型的返回值。


## 函数的原型 {id="prototype"}

在 C 语言中，我们一般会为函数定义原型，原型就是没有函数体的函数声明。比如:
```c
int sum(int, int);
// 或者
int sum(int left, int right);
```
函数原型的意义，在于告诉我们的编译器函数的相关声明，具体的函数实现会在其他的文件或库中。但是现代的一些编译器并不需要函数的原型。


## 变长参数 {id="variable-arguments"}

在 C 语言中，在函数中也支持变长的参数传入。但相对于其他高级语言来说，就显得比较原始，是使用一些宏来实现的。下面这个例子使用了变长参数来求和:
```c
#include <stdarg.h>

void sum(int size, ...) {
    int sum = 0;
    va_list args;
    va_start(args, size);
    for (int i = 0; i < size; i++) {
        sum += va_arg(args, int);
    }
    printf("%d", sum);
    va_end(args);
}
```
我们来测试这个函数, 和我们预期的结果应该是一样的，输出是 15:
```c
int main() {
    sum(5, 1, 2, 3, 4, 5);
}
```
在这段代码中，我们从 `<stdarg.h>`头文件中引入了三个宏来帮助我们获取变长参数: `va_list`、 `va_start`以及 `va_end`。这要按照这种固定的形式写就好了。


## 递归 {id="recursion"}

我们把函数自己调用自己称之为递归，在满足条件后停止自我调用。下面我们分别演示使用递归和不使用递归来求解 fibonacci 数列。

什么是 Fibonacci 数列呢？就是当 n >  2 的时候，后面一个值等于前面两个值的和。下面是一组 Fibonacci 的数列:

| 0 | 1 | 1 | 2 | 3 | 5 | 8 | 13 | 21 | 23 | 55 | 89 |
|---|---|---|---|---|---|---|----|----|----|----|----|


### 使用循环求解 {id="fibonacci-by-loop"}

首先我们先使用循环求解:
```c
unsigned int fibonacci(unsigned int num) {
    // 如果是 0 和 1 就直接返回，是固定的
    if (num == 0 || num == 1) return num;
    unsigned int left = 0, right = 1;
	for (unsigned int i = 2;i <= num;i++) {
        // 保存右边的值，需要参与第二次计算
		unsigned int temp = right;
        // 求和
		right = left + right;
        // 右边的值赋值给左边的值
        left = temp;
    }
	return right;
}
```

### 使用递归求解 {id="fibonacci-by-recursion"}

使用递归看上去要简单很多，但是递归对内存的消耗是巨大的:
```c
unsigned int fibonacci(unsigned int num) {
    // 终止条件
    if (num == 0 || num == 1) {
        return num;
    }
    // 求和
    return fibonacci(num - 1) + fibonacci(num - 2);
}
```

## 总结 {id="summary"}

这篇文档介绍了 C 语言中，函数的使用，并且分别使用递归和循环两种方式求解 Fibonacci 数列。
