# 列表和元组

定义一个元组:

```python
tup = ('Hello', 3, ['Hello', 3])
```

定义一个列表:

```python
li = ['Hello', 3, ['Hello', 3]]
```

元组和列表的相同点:

- 元组和列表都可以包含一组数据
- 元组和列表包含的一组数据中，都不限定数据类型
- 元组和列表的元素的访问方式都是一样的，可以通过 `tmp[index]` 这样的方式访问

元组和列表的不同点:

- 在语法上，元组使用 `()` 定义，而列表使用 `[]` 定义
- 列表是动态的，其长度大小是可以改变的，可以新增元素和修改元素。这是元组所不能的，所以元组是静态的。

如果改变元组的元素，则 Python 会给与错误提示:

```python
>>> tup = ('Hello')
>>> tup[0] = 'World'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
```

在 Python 中，字符串、列表、元组都支持负索引，如下:

```python
>>> str = 'Hello World'
>>> str[-1]
'd'
>>> li = ['Hello', 'World', '!']
>>> li[-1]
'!'
>>> tup = ('Hello', 'World', '!')
>>> tup[-1]
'!'
```

另外，列表和元组是可以互换的:

```python
>>> tup = tuple(['Hello', 'World'])
>>> print(tup)
('Hello', 'World')
>>> li = list(('Hello', 'World'))
>>> print(li)
['Hello', 'World']
```

获取列表或元组的个数:

```python
>>> tup = ('hello', 'world')
>>> len(tup)   # 元素的个数
```

列表元素反转:

```python
>>> li = ['Hello', 'World']
>>> li.reverse()
>>> print(li)
['World', 'Hello']
```

获取元素出现的次数:

```python
>>> li = ['a', 'b', 'c', 'a']
>>> li.count('a')
2
```

获取元素首次出现的索引:

```python
>>> li = ['a', 'b', 'c', 'a', 'd']
>>> li.index('d')
4
```

列表和元组在内存中的占用空间是不一样的, 很明显元组要占用更小的空间:

```python
>>> li = [1, 2, 3]
>>> tup = (1, 2, 3)
>>> li.__sizeof__()
64
>>> tup.__sizeof__()
48
```

下面的例子展示他们之间的性能也是有差异的, 元组比列表大约快 3.4 倍:

```shell
$ python3 -m timeit 'x=(1,2,3,4,5,6)'
10000000 loops, best of 3: 0.0539 usec per loop

$ python3 -m timeit 'x=[1,2,3,4,5,6]'
10000000 loops, best of 3: 0.187 usec per loop
```

因为，如果元素都是不变的，那么应该使用元组，而元素如果会发生变化，应该使用列表。