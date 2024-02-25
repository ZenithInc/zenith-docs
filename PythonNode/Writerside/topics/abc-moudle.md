# Abc模块和抽象基类

面向对象的三大特性是继承、封装和多态，其核心是抽象。通过抽象建立层次结构，分清楚职责，管理依赖。

在实际的业务中，我们经常要提取父类、抽象接口。**但是在 Python 中并没有接口一说，接口主要起到一个约束的作用**。但是在 Python 中可以使用 abc 模块和抽象基类来实现类似的功能, 约束继承的类，或者实现的类必须实现特定的方法。

> 什么是接口? 举个例子，你电脑上都会有 USB 接口，但是你一定要把网线的 RJ45 接口塞到 USB 接口中，那么要么网线坏了，要么 USB 接口坏了。


## 使用继承 {id="extends"}

比如我们有一个 BaseService 的基类，它有两个或者更多的实现类，比如 UserService 以及 ProductService......我们希望实现类可以像接口一样去实现基类中所有的方法。我们可以使用继承的形式来完成它。

```python
class BaseService:
    def foo(self):
        raise NotImplemented()
    def bar(self):
        raise NotImplemented()
class UserService(BaseService):
    def foo(self):
        print('foo')
user = UserService()
user.foo()
user.bar()  ## 抛出异常: raise NotImplemented()
```

上面的代码中，我们定义了两个类，分别是 BaseService 和 UserService, 其中 UserService 继承 BaseService。接着我们分别调用 `user.foo`和 `user.bar`，但是在调用 `user.bar()`的时候报错了, 抛出了 `NotImplemented()`异常。

这种继承的方式勉强实现了如果没有全部实现基类的话就报错的功能，但是只有在调用 `user.bar()`的时候才会抛出异常，实例化 `UserService()`以及调用 `user.foo()`的时候并不会抛出异常。

所以这个实现方案是不那么完美的，我们希望它能够更早的抛出异常，在实例化的时候就抛出异常。所以，我们要引入 abc 模块和抽象方法的概念。

## abc 模块和抽象方法 {id="abc"}

上文我们说到，纯粹使用继承的方式，并不能更早的发现错误。而且它和接口不一样，并不是在语言层面抛出的错误，而是我们自己在基类的方法中通过硬编码的形式抛出的异常。

接下来，我们使用 abc 模块和抽象方法来重写上面的示例:

```python
from abc import ABCMeta, abstractmethod 

class BaseService(metaclass=ABCMeta):
    @abstractmethod
    def foo(self):
        pass
    @abstractmethod
    def bar(self):
        pass
class UserService(BaseService):
    def foo(self):
        print('foo')
        
user = UserService()
```

首先我们在代码的第一行引入了 abc 模块中的 `ABCMeta` 类，和 `abstractmethod` 注解。然后，我们在代码的第 4 行和第 7 行分别是使用 `abstractmethod` 注解来表明 `foo` 和 `bar` 两个方法是抽象方法。

最后，我们去实例化 `UserService()` 的时候就会报错，因为我们并没有实现基类中的 `bar` 方法:

```
user = UserService()
TypeError: Can't instantiate abstract class UserService with abstract methods bar
```

错误的意思就是 UserService 中必须要包含 bar 这个方法。

## 总结 {id="summary"}

这篇文档概述了面向对象编程中的三大核心特性：继承、封装、多态，并强调了抽象的重要性。在Python中，虽然没有传统意义上的接口概念，但可以通过`abc`模块和抽象基类实现类似接口的功能，以约束继承自抽象基类的子类必须实现特定的方法。

文档通过一个比喻说明了接口的概念，然后通过具体的代码示例展示了如何在Python中使用继承来尝试实现接口的功能。但是，仅仅使用继承并不能完全模拟接口的行为，因为它不能在实例化时就发现未实现方法的错误。

随后，文档引入了使用`abc`模块和抽象方法来改进这一点，通过定义抽象基类和抽象方法，可以在尝试实例化未完全实现抽象方法的类时，立即抛出错误。这种方式更贴近于接口的原始意图，即在编译或解释阶段就能发现潜在的错误，提高了代码的健壮性和可维护性。