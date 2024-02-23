# 字符串

在实际的场景中，我们需要经常对字符串类型的数据进行各种的处理。比如截取字符中的子串，比如计算字符串的长度等。所以，这一篇文档将对字符串的处理加以讲解。

## 计算字符串的长度 {id="string-length"}

对字符串的长度进行计算，有两种做法，如下示例:
```bash
str=Hello
echo ${#str} ## 输出: 5
```
第二种做法如下:
```bash
str=Hello
expr length "$str" ## 输出: 5
```
> 需要注意的是，字符串的变量都需要加上引号，不然在字符串中存在空格的情况下会出现意想不到的错误。


## 获取字串在字符串中的索引位置 {id="string-index"}

我们再看看如何获取指定的字串在字符串中的位置:
```bash
str='Hello World'
expr index "$str" Hello  ## 输出: 1
```
然后，再看看下面这个例子:
```bash
str='Hello World'
expr index "$str" Word ## 输出: 5
```
为什么是输出为 3 呢？ Word 这子串在 “Hello World”字符串中并不存在啊。其实， `expr index`  并不是去寻找字符串的位置，而是将 `Word`  字符串拆分成单个字符，即 `W` 、 `o` 、 `r` 、 `d` ，然后分别取查找这几个字母在 “Hello World”字符串中的位置，结果找到了 `o` 在 “Hello World”字符串中的位置为 5。

## 获取字串的长度 {id="expr-match"}

先看下面的示例:
```bash
str='Hello World'
expr match "$str" World ## 输出: 0
expr match "$str" Hell ## 输出: 4
```
World 的输出为什么是 0 呢？因为 `expr match` 必须是从头开始匹配，否则就是 0。

## 抽取字串 {id="find-sub-strings"}

从指定字符串中抽取字串有很多方法，大体如下表所示:

|     | 语法                                      | 说明                         |
|-----|-----------------------------------------|----------------------------|
| 方法一 | `${string:position}`                    | 从 string 中的 position 开始    |
| 方法二 | `${string:position:length}`             | 从 position 开始,匹配长度为 length |
| 方法三 | `${string: -position}`                  | 从右边开始匹配                    |
| 方法四 | `${string:(position)}`                  | 从左边开始匹配                    |
| 方法五 | `expr substr $string $position $length` | 从 position 开始，匹配长度为 length |

然后我们来演示一下:
```bash
str='Hello World'
echo ${str:5} ## 输出: World
echo ${str:2:4} ## 输出: llo
echo ${str: -5} ## 输出: World, 注意':' 后面一定要加上空格
echo ${str:(-5)} ## 输出: World, 如果使用()，则 : 后面不需要加空格
echo ${str:(-5):2} ## 输出: Wo, 指定截取长度
expr substr "$str" 2 5 ## 输出: ello
```
## 总结 {id="summary"}

这一篇文档主要描述了 Shell 脚本编程中对于字符串的相关处理。