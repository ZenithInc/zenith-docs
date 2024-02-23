# 字典和集合

字典的定义如下:

```python
>>> dict = {'name': 'JinZhiSu', 'job': 'developer'}
>>> dict['name']
'JinZhiSu'
>>> dict['job']
'developer'
```

字典是一组键值对(key/value), 如果是没有定义的 key 呢？Python 会给出错误提示:

```python
>>> dict['undefined']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'undefined'
```

如果不希望程序中断执行，可以使用 `get` 方法，如果不存在则会返回 `None`

```python
>>> print(dict.get('undefined'))
None
```

集合和字典大体上是一样的，只是字典是键值对的组合，而集合是纯值得组合:

```python
>>> s = {1, '2', '3', '1'}
>>> print(1 in s)
True
>>> print(2 in s)
False
```

集合和字典都是支持增加和移除的:

```python
>>> s = {1, 2}
>>> s.add(3)
>>> print(s)
{1, 2, 3}
>>> s.remove(3)
>>> print(s)
{1, 2}
>>> dict = {'name': 'JinZhiSu', 'email': "happy@hacking.icu"}
>>> dict['age'] = 12
>>> print(dict)
{'name': 'JinZhiSu', 'email': 'happy@hacking.icu', 'age': 12}
>>> dict.pop('name')
'JinZhiSu'
>>> print(dict)
{'email': 'happy@hacking.icu', 'age': 12}
```

需要注意的是，从 Python3.7 开始，字典是有序的，而集合仍旧是无序的。所以，要慎重使用 `s.pop()` 操作，因为集合是无序的，所以也不知道他会移除什么元素。