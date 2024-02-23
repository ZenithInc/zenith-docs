# 语法快速入门

这篇文档从环境讲起，然后快速入门 PHP 的语法。不会覆盖所有的语法，够用就好。

## Playground {id="playground"}

搭建 PHP 的开发环境有一些麻烦，最好的方式就是利用在线的 Playground 直接上手写代码。我推荐使用 [PHPPlayground](https://www.phplayground.com/) 这个网站，可以选择最新的版本
进行实验，如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/iv2BLZVKS9b315HvConD.png" alt="php playground"/>

这一系列的教程基于目前最新的 PHP 8.2 版本来编写。

## Hello World {id="hello-world"}

首先我们来写一个 `Hello World` 的程序开启 PHP 之旅:
```PHP
<?php
echo 'Hello World'
?>
```
运行这个示例，会在屏幕上输出`Hello World` 。在实际的开发中，我们会将最后一行 `?>` 省略。但是第一行`<?php` 不能省略，这是所有 PHP 代码的第一行。

第二行 `echo 'Hello World';` 中 `echo` 表示这个单词的意思是“回响”，意思就是将 `Hello World` 这句话打印在终端。


## 变量和常量 {id="php-variables-and-const"}

上面的简单示例中，我们把数据“硬编码“到了代码语句中，`echo 'Hello World';` 中的 `Hello World` 就是数据。在良好的程序设计中，我们需要将数据和编码分离，这样有利于数据的更新和复用。于是就有了**变量和常量的概念，区别在于前者可变，后者不可变。**

如果你觉得 `Hello World` 在代码中永远都不会发生变化，你就可以将它声明为常量,使用`const` 关键字:
```PHP
const GREETING = 'Hello World';
echo GREETING;
```
当然，如果你预期 `Hello World` 这个数据可能会变为`Hello PHP`, 这时候就应该将它声明为变量，使用 `$` 作为变量名的前缀:
```PHP
$greeting = 'Hello World';
echo $greeting;
```
当我们将一个数据通过`=` 赋值给一个常量或者变量的时候，我们称这样的语句为“赋值语句”。

<note>
你可能已经发现了，变量我们的名称我们使用的是小写字母，而常量我们使用的是大写字母。在 PHP 语法中，并没有强制规定，但这是大部分 PHP 开发者都遵守的规则。
</note>

数据根据使用场景的不同，我们划分了一些基本类型, 这是符合我们的生活常识的，比如生活中我们也会区分小数和字符语句。

| 类型               | 描述                   | 示例                 |
|------------------|----------------------|--------------------|
| `string`         | 字符串类型                | `Hello World`      |
| `int`            | 整型                   | -1、0、1、2、3...      |
| `float`、`double` | 浮点类型/双精度类型,暂时可以理解称小数 | 0.00， 1.29         |
| `bool`           | 布尔类型                 | true 表示真，false 表示假 |

还有一些其他类型，我们遇到了再说。`bool` 类型比较特殊，取值范围只有两个值，分别是 `true` 和 `false`。有些开发者也会写为大写，但一般都使用小写。

## 分支 {id="select"}

在生活中处处有选择，代码是现实生活的映射。所以也处处有选择。

### if...else... {id="if-else"}
在 PHP 中，使用 `if...else..`语句，和其他语言没有什么区别:

```PHP
$name = 'boss';
if ($name === 'boss') {
    echo "Woking!"; // 这句将会执行
} else {
    echo "Play game!";  // 这句不会执行
}
```
> `//` 表示单行注释，是对代码的解释，不会执行。写出合理的注释，是良好的编程习惯。偶尔也有人将`//` 写成 `#` ，这也可以，只是很少有人这么写。

也支持多分支的写法，如下:
```PHP
$weather = 'sunny';
if ($weather === 'rainy') {
    echo 'At home';
} else if ($weather === 'sunny') {
    echo 'Go out!';
} else {
    echo "I don't known";
}
```
> 可以看到，上面代码中有的字符串字面量使用`'` 单引号，有的使用 `"` 双引号。在大多数情况下，使用单引号就好了，但是如果字符串中包含特殊字符`'` 或者包含变量，则可以使用双引号。

### switch {id="switch"}

有的场景下，分支比较多，使用 `switch` 语法可能会更加简洁。
```PHP
$today = 'Sunday';
switch ($today) {
    case 'Monday':
        echo 'Go Shoping';
        break;
    case 'Thuesday':
        echo 'Sleeping';
        break;              // 匹配后推出 switch
    case 'Wednesday':
        echo 'Learning';
        break;
    default:
        echo 'Working';
}
```
> 需要注意的是，每一个 `case` 之后都需要加上 `break`，如果没有特殊理由希望它继续匹配执行的话。所以有的语言里，干脆没有 `switch` 语法了。

### match 表达式 {id="match-expression"}

这个是 PHP8 后出现语法, 可以简化 `switch` 或者 `if...else..` 的编写:
```PHP
$operater = '-';
$result = match ($operater) {
   '+' => 1 + 1,
   '-' => 1 - 1,
   '*' => 1 * 1,
   '/' => 1 / 1,
};
echo $result;
```