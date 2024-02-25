# 探究 finally

在 Python 中，为了避免资源在异常情况下没能释放，所以有了在 `try`语句块的后面加上 `finally`的做法。不管代码是否发生异常，都会执行 `finally`的语句块。

但是如果我们继续深究 `finally`，会有一些细节可能被我们忽略了。所以，本文希望可以挖掘出这些细节，避免因为对他们的忽略导致一些难以排查的问题的出现。

我们用如下的示例代码来解答一些问题:

```python
def test():
    try:
        print("a")
        return 0
    finally:
        print("b")
        return 1

print(test())
```

根据这段代码以及运行的结果，我们来回答如下问题:

1. 可以在 `finally`语句块中加入 `return`吗？可以的，运行这段代码不会出现任何错误。
2. 第 4 行的 `return`和第 `7`行的 `return`会执行哪个? 执行第 7 行的，所以第 9 行打印的结果是 1。

上面的代码也证实了其实 `finally`会在第 4 行的 `return`之前运行。然后我们再来看下面这段示例代码:

```python
li = [1, 2]
def test():
    try:
        return li
    finally:
        li.append(3)
        
print(test())
```

打印的结果是什么？是的，是 `[1, 2, 3]`。因为上文中我们已经说过了 `finally`语句块会在 `return` 之前执行。那么我们再来看看下面这段代码:

```python
str = "Helo World"
def test():
    try:
        str = "Hello Cat"
        return str
    finally:
        str = "Hello Python"

print(test())
```

打印的结果是什么？不是的，而是 `Hello Cat`！不是说 `finally`会在 `return` 之前运行吗？而且在上一个示例中， `li` 的值确实发生了变化啊。这是为什么？

因为在 `return`之前，程序会暂存 `str` 的结果，之后去调用 `finally` 的语句。而 `str`是  String 类型的，是值类型，所以是拷贝了一份，在 `finally` 中并的更改并没有生效。而之前的 `li` 变量之所以被更改了，是因为它是 List 类型的，在 Python 中， List 类型是引用类型，所以可以在 `finally`中被更改。