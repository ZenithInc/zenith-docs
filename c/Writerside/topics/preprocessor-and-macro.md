# 预处理和宏

在之前的文档中，对于编译都是一语以蔽之，并没有展开来讲。在 C 语言中，有两个概念是需要理解的，分别是预处理和宏，他们分为处于预处理阶段和编译阶段。接下去，我们就来说说它们。

## C 语言程序的编译过程 {id="how-to-compile-a-c-program"}

一个 C 语言的程序，其生命周期如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/LErPYOAxBhD239ZxMGEU.png" alt="how to compile a c program"/>

1. **编写源代码**。这个时候，我们的代码文件和其他的文件(比如我写的文章)并没有什么区别。
2. **预处理器处理我们的代码**。预处理器也是一个软件，集成在了类似于 gcc
   这样的编译套件中了。它会做一些少量的工作，比如将我们使用 `#include、#define`这样的预处理命令替换成实际的内容。
3. **编译器编译代码**。将我们 C 语言的代码编译成汇编语言的对象文件。
4. **汇编器将我们的代码转换成机器代码。**
5. **链接器将中间文件、库链接在一起，生成可执行文件。**
6. **我们基于 GUI 或者 CLI 来执行可执行文件**，程序运行直到终止。

### 文件引入 {id="include-file"}

理论上，我们可以把所有的代码都写在一个文件中。但是，这就好像超市把所有的商品都排布在一个货架上一样，难以管理。所以，需要有一种机制，将不同文件中的代码汇聚到一起(
换一个角度来说，是将原本写在一起的代码分布到不同的文件中去)。我们可以使用 C 语言当中的 `#include` 指令。

不管是用户定义的还是系统定义的头文件，都可以使用 `#include` 指令引入并展开, 它有两种语法:

```c
#include <stdio.h>
#include "my_header.h"
```

这两种语法的区别是：使用 `<>`一般指的是引入系统的头文件，而使用 `""`指的是引入自定义的或者第三方的头文件。**
其实，不管是使用 **`**<>**`**还是使用 **`**""**`**都可以引入系统头文件或者自定义的头文件。区别是使用引号会首先查找当前目录中是否存在该头文件，会有一定的资源开销。
**

我们前文中说到，使用 `#include`指令是引入和展开，什么是展开呢？意思就是使用指定头文件中的内容替换 `#include`指令所在行。

创建一个三个文件，如下:

- **main.c**: 作为项目的入口文件，存放入口函数- `main`
- **hello.h**: 头文件，写一些函数的原型，告诉编译器函数的入参和出参
- **hello.c**: 源文件，实现在 hello.h 文件中的原型

main.c 文件内容如下:

```c
#include "include/hello.h"

int main() {
    PrintHello();
    return 0;
}
```

接下来实现 `include/hello.h` 文件, 需要注意的是 `#ifndef` 这样写法是条件编译，暂时可以不用理会:

```c
#ifndef CDEMO_HELLO_
#define CDEMO_HELLO_

#include <stdio.h>
void PrintHello(void);

#endif
```

最后实现`src/hello.c`文件:

```c
#include "../include/hello.h"

void PrintHello(void){
    printf("Hello World!\n");
}
```

## 宏定义的妙用 {id="macro"}

在 C 语言中，宏有很多的用处。我们将在下文中一一列举这些用处，让大家对宏有一个新的认识。

### 使用宏来定义常量 {id="defined-const"}

这个在前文中也已经讲到了，我们可以使用宏在定义常量。使用宏定义的常量，会在预处理阶段，使用真实的常量值替换到代码中用到的宏。我们举例说明:

```c
#define TRUE 1
#define FALSE 0
int main(){
    printf("%d %d\n", TRUE, FALSE);
}
```

### 使用宏来定义函数 {id="defined-function"}

我们可以使用宏来定义一些简单的函数，在预处理阶段，代码中的宏就会被替换成定义的函数。这样即利用了函数的封装性、提升了代码的可读性，又避免了函数进栈出栈的开销。如下示例:

```c
#include <stdio.h>

#define MAX(left, right) left > right ? left : right

int main() {
    printf("3 compare 5: %d\n", MAX(3, 5));     // Output: 5
    printf("13 compare 5: %d\n", MAX(13, 5));   // Output: 13
    return 0;
}
```

但是使用宏函数的时候一定要注意，有一些可能的错误。就那上面这个 `MAX`宏函数来说，测试如下:

```c
int result = MAX(2, MAX(3, 2));
printf("result: %d\n", result);     // Output: 2
```

我们设想先对比 3 和 2，这个结果在和 2 对比，所以正确的结果应该是 3。但是输出的结果是 2
这是为什么呢？因为宏函数只是简单的字面替换并没有优先级，上面的宏最终被替换成了如下形式:

```c
2 > 3 > 2 ? 3 : 2 ? 2 : 3 > 2 ? 3 : 2
```

怎么避免这样的情况的发生呢？我们需要使用 `()`来将宏函数的每一个参数都标记为表达式，如下:

```c
#define MAX(left, right) (left) > (right) ? (left) : (right)
int result = MAX(2, MAX(3, 2));
printf("result: %d\n", result);     // Output: 3
// 展开替换后如下形式
int result = (2) > ((3) > (2) ? (3) : (2)) ? (2) : ((3) > (2) ? (3) : (2);
```

我们也可以改成多行形式的函数，比如下面定义了一个判断是否为十六进制合法字符的一个宏函数:

```c
#define IS_HEX_CHARACTER(ch) \
((ch) >= '0' || (ch) <= '9') || \
((ch) >= 'A' || (ch) <= 'F') ||  \
((ch) >= 'a' || (ch) <= 'f')
```

> 注意: 定义的时候换行是为了代码的可读性，但是在替换之后是不会换行的。

### 使用宏来实现条件编译 {id="conditional-compilation"}

宏可以被用作条件编译，在本文之前的章节中已经出现过了，只是没有详加解释，如下:

```c
#ifndef CDEMO_HELLO_
#define CDEMO_HELLO_

void PrintHello(void);

#endif
```

如果在代码中类似于 `PrintHello`
这样的函数原型被重复定义，那么编译是会出错的。为了避免出现这样的问题，我们可以使用 `#ifndef` 和 `#endif` 这样的条件编译指令。

如上示例，编译器在编译的时候会检查 `CDEMO_HELLO_` 这个宏有没有被定义，如果没有定义则会编译`#ifndef` 和 `#endif`
之间的代码，否则不会编译。在这个当中，我们在 `#define` 这个宏。这样就有效避免了对函数原型的重复定义。

除了 `#ifndef` 之外，还有  `#ifdef`， 两者是相反的，如果定义了指定的宏则编译。比如说，我们在不同的操作系统中会有不同的宏定义，我们可以借此来区分不同的
OS，以执行不同的代码。

```c
// 判断是否处于 C++ 环境
#ifdef __CPLUSPLUS
// 判断 C 标准版本
#if __STDC_VERSION__ >= 201112
	printf("11版本");
#elif __STDC_VERSION__ >= 19901
	printf("90版本");
#else
	printf("Other version");
// 判断运行操作系统
#ifdef __UNIX__
#ifdef _WINDOWS
```

或者我们可以使用 `#if` 来指定表达式，比如说打印 DEBUG 信息:

```c
#if defined(DEBUG)
printf("XXXX");
#endif
```

### 宏函数和普通函数的区别 {id="difference-macros-between-functions"}

那么宏函数和普通的函数有什么区别呢？我们把它们之间的区别总结在下面这张表格中:

| 对比项    | 宏函数                                                          | 普通的函数                                                 |
|--------|--------------------------------------------------------------|-------------------------------------------------------|
| 代码长度   | 每次使用时，宏代码都会被插入到程序中。除了非常小的宏之外，程序的长度会大幅增加。                     | 函数代码只出现在一个地方。每次使用这个函数时，调用用那个地方的一份代码。                  |
| 操作符优先级 | 宏参数的求值在所有周围表达式的上下文环境里，除非加上括号，否则临近的操作符优先级可能会产生不可预料的结果。        | 函数参数只在函数调用时求值一次，他的结果传递给函数。表达式的求值结果更容易预测。              |
| 参数求值   | 参数每次用于宏定义时，他们都将重新求值。由于多次求值，具有负作用域参数可能会产生不可预料的结果(比如说param++)。 | 参数在函数被调用前求值一次，在函数中多次使用并不会导致多次求值的问题，参数的负作用不会造成任何特殊的问题。 |
| 参数类型   | 宏与类型无关。只要对参数的操作是合法的，就可以适用于任何参数。                              | 函数的参数是与类型有关系的。如果参数的类型不同就需要使用不同的函数，即使他们执行的任务是相同的。      |

综合上面这张表格所示，**使用宏函数要比使用普通的函数更加的小心谨慎。**

### 使用宏函数实现打印 DEBUG 信息 {id="print-debug-info"}

在开发过程中，我们经常会使用 `printf`
这个函数来打印一些调试信息，但是这个函数并不是那么好用，比如说我每次都要在后面加上一个 `\n` 换行的转义字符。

我们可以借助宏来实现优化后的 `printf` 函数，来方便我们输出调试信息。

```c
// 实现换行
#define PRINTLNF(format, ...) printf(format"\n", __VA_ARGS__)
// 测试
PRINTLNF("%s %s", "Hello", "World");
```

你发现了没有，宏函数也支持变长参数，使用 `...` ,那么如何获取变长参数呢？可以使用另一个宏 `__VA_ARGS__` 。

但是这个宏的实现，还有一点点的缺陷，那就是变长参数至少要有一个值，否则替换后就会多一个逗号，导致编译错误。为了解决这个问题，可以使用 `##` ,
如下示例:

```c
#define PRINTLNF(format, ...) printf(format"\n", ##__VA_ARGS__)
// 测试
PRINTLNF("Hello World");
```

在实际开发中，我们经常需要同时输出变量名和变量值，但是在代码中并不能获取变量的名字啊，所以需要硬编码实现。这个问题也可以通过宏来解决:

```c
#define PRINT_INTEGER(value) PRINTLNF(#value": %d", value)
// 测试
int num = 10;
PRINT_INTEGER(num); // Output: num: 10
```

还有一种场景，我希望打印文件名、函数名以及行号，也可以实现，如下:

```c
#define PRINT_POSITION(message) PRINTLNF("("__FILE__":%d) %s: %s", __LINE__, __FUNCTION__, #message)
// 测试
PRINT_POSITION("Hello World"); 
// Output: (/Users/sujinzhi/CLionProjects/CDemo/main.c:16) main: "Hello World"
```

## 参考资料 {id="reference"}

1. [C/C++语言编译链接过程-知乎](https://zhuanlan.zhihu.com/p/88255667)
2. [Include Syntax - The C Preprocessor](https://gcc.gnu.org/onlinedocs/cpp/Include-Syntax.html)