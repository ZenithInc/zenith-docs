# With: 上下文管理器

在一些代码中，经常能看到 With 这个语法的身影，这是什么意思呢？他是一种语法糖，上下文管理器。这篇文档就用以描述清楚什么是上下文管理器，With 如何使用。

首先，来演示第一种使用类的写法，当我们在一个类中声明两个方法，分别是 `__enter__` 和 `__exit__` 。就可以使用 `with` 语句。

```python
class Resource:
    def __init__(self):
        print('资源类的初始化')
        
    def exec(self):
        print('针对资源做你想做的事情')
    
    def __enter__(self):
        print('获取指定的资源')
        
    def __exit__(self):
        print('释放指定的资源')
```

然后我们就可以针对上面这个类使用 `with` 语法:

```python
with Resource() as r:
    r.exec()
```

你看看它输出的顺序是什么？