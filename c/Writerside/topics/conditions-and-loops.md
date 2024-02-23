# 分支和循环 

现在大部分的高级语言，在分支和循环语句上，都采用了类似于 C 语言的语法。所以 ，C 语言种的分支和循环是比较经典的语法形式。为了让这系列的文档更加的完整，所以有了这一篇非常“枯燥”的分支和循环。

不管对于作者还是读者来说都是“枯燥”的，因为在其他的语言中，基本都是一样的。所以，本文不会举一些特殊的例子，基本上将语法一笔带过而已。

## 分支 {id="conditionals"}

在 C 语言中，分支语句有三种写法，分别是 `if...else...` 、 `switch` 和三元运算符（ `?:` )。下面我们分别来介绍这三种语法:
```c
if (conditions) {
    statements
} else if(conditions) {
 	statements   
} else {
    statements
}
```
第二种是采用 `switch` :
```c
switch (expression) {
    // case 后面也可以使用 {} 来代替:
    case expression:
        statements
        break;
    default:
        statements
}
```
第三种是三元运算符:
```c
expression ? statement : statement;
// 举例:
a > 10 ? printf("gt") : printf("lt")
```

### 循环 {id="loops"}

在 C 语言中，循环有几种语法结构: `while` 、 `do while` 、 `for` 。下面分别来介绍这几种语法形式:
```c
while(conditions) {
 	statements
}
```
下面是 `do while` ，区别于 `while` , 先执行一遍循环体:
```c
do {
    statements
} while (conditions);
```
最后是 `for` ，特别适合用来遍历数组这样的数据结构:
```c
for (<initialization>;<conditions>;<state>) {
 	statements   
}
```
需要注意的是在 C99 标准以前，是不支持在 `for` 语句中定义变量的，之后才支持:
```c
// C99 以前
int i;
for (i = 0; i < 10; i++) {}
// C99 以后
for (int i = 0; i < 10; i++) {}
```
另外，也可以在语句结构中定义多个变量，如下:
```c
for (int i = 0, j = 0;  i < 10 && j < 10; i++, j++) {}
```

## 总结 {id="summary"}

这篇文档介绍了 C 语言中分支和循环的写法。