# 六大原则之里氏替换原则

里氏替换原则（Liskov Substitution Principle，LSP）是由麻省理工学院计算机科学系教授芭芭拉 · 利斯科夫（Barbara Liskov）在 1987 年的 “面向对象技术的高峰会议”（OOPSLA）上发表的一篇文章《数据抽象和层次》（Data Abstraction and Hierarchy）里提出的。

![LSP-Wiki](http://file-linker.oss-cn-hangzhou.aliyuncs.com/9k75KCe0zuDBoTwxRiFI.png)

## 什么是里氏替换原则

你从上面的截图中也应该看到了，里氏替换原则的原文是一种数学的表达，并不好理解。原文如下:

> "Let ϕ(x) be a property provable about objects x of type T. Then ϕ(y) should be true for objects y of type S where S is a subtype of T."


如果就字面翻译，其意思是: 如果对于类型 T 的对象 x，有一个属性 ϕ(x) 被证明为真，那么对于 T 的子类型 S 的对象 y， ϕ(y) 也应该被证明为真。

如果你和我一样还是看不懂其在说什么，我们可以再翻译为更容易理解的版本: **如果一个程序使用了父类的对象，那么替换为子类的对象后，程序的行为没有变化，子类必须能够替换掉它们的父类并且正确地工作。**

如果看上去字数还是有点多，可以再缩句为：**子类可以扩展父类的功能，但不能改变父类原有的功能，使得子类能够无缝替换父类**。具体来说，要遵循下面这些规则：

- 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。
- 子类可以增加自己持有的方法。
- 当子类的方法重载父类的方法时，方法的前置条件（即方法的输入参数）要比父类的方法更宽松。
- 当子类的方法实现父类的方法（重写、重载或实现抽象方法）时，方法的后置条件（方法的输出或返回值）要比父类的方法更严格或与父类的方法相等。

## 电动汽车的例子

我们来看下面这个电动汽车的例子，其违反了 LSP：

```java
class Vehicle {
    public void refuel() {
        System.out.println("Refueling the vehicle");
    }
}

class ElectricCar extends Vehicle {
    @Override
    public void refuel() {
        throw new UnsupportedOperationException("Electric cars do not support refueling");
    }
}

class Main {
    public static void main(String[] args) {
        Vehicle myCar = new ElectricCar();
        myCar.refuel(); // 这里会抛出UnsupportedOperationException异常
    }
}
```

这真是一个有趣的例子。早些年没有电动汽车，所以可以为 `Vehicle` 的基类添加一个 `refuel` 的方法。但是后来出现了电动车，因为电动车不需要加油, 这样的设计就需要重构：

```java
abstract class Vehicle {
    public abstract void drive();
}

abstract class FuelVehicle extends Vehicle {
    public void refuel() {
        System.out.println("Refueling the vehicle");
    }
}

class ElectricCar extends Vehicle {
    @Override
    public void drive() {
        System.out.println("Driving electric car");
    }
    
    public void recharge() {
        System.out.println("Recharging electric car");
    }
}

class GasCar extends FuelVehicle {
    @Override
    public void drive() {
        System.out.println("Driving gas car");
    }
}
```

我们 `Vehicle`拆分为 `FuelVehicle`（油车），`ElectricCar`继承 `Vehicle`，而 `GasCar`继承 `FuelVehicle`。就解决了电车加油的问题，其 UML 如下：

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/2ISYJR8bo41CuMV5x1KU.png)

但是这种深层次的继承并不推荐，因为一旦出现需求上的变更，想要改变继承的层次结构是非常困难的。所以我们常说**组合由于继承**，更推荐使用接口的方式来实现这个重构，想要改变实现的接口是容易的。且接口支持多实现，而大多数语言中，不支持多继承。即使支持，也很难用好。

```java
interface Vehicle {
    void drive();
}

interface FuelVehicle extends Vehicle {
    void refuel();
}

class ElectricCar implements Vehicle {
    public void drive() {
        // 实现驾驶逻辑
    }

    public void recharge() {
        // 实现充电逻辑
    }
}

class GasCar implements FuelVehicle {
    public void drive() {
        // 实现驾驶逻辑
    }

    public void refuel() {
        // 实现加油逻辑
    }
}
```

使用接口代替了原本的抽象类，其 UML 如下:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/ajZR5XDaO8Cn6uh5bDFU.png)

类似的例子还有很多，比如 “鸵鸟非鸟”、“正方形不是长方形”等。这个例子说明了继承能够表示的是一种 `is-a`的关系，但是父类和子类紧密耦合在一起，难以适应未来需求的变化，特别是深层次的继承关系。

相较而言，使用接口更加灵活。但是并不意味着使用接口就不需要关注 LSP。LSP 虽然说的是继承关系，但同样适用于接口。在设计代码的时候，我们需要注意一下两点，避免违反 LSP：

- 里氏替换原则是告诉我们在**使用继承的时候，一定要确保子类 is a 父类，并且拥有父类的所有能力。**
- 当我们**使用接口的时候，确保所有的实现都能符合接口的契约**。

## 总结

本文中我们讨论了其定义、重要性、实现方法以及它在现代软件开发中的应用。以下是对这些讨论内容的总结：

**里氏替换原则的定义**: 里氏替换原则是面向对象设计原则之一，强调子类对象应该能够替换其父类对象，而不改变程序的正确性和行为。这意味着，设计子类时，不仅要保持接口的一致性，还要确保行为的兼容性。

**里氏替换原则的重要性**如下:

- **提高软件的可维护性**: 遵守 LSP 有助于保持代码的一致性和预测性，使得软件更容易维护和理解。
- **增强软件的可扩展性**：符合 LSP 的设计允许系统轻松扩展，通过添加新的子类增加功能，而不需改动现有代码。
- **减少耦合**：LSP 鼓励使用抽象和接口，减少了类之间的紧密耦合，使得各个部分更加独立，易于修改和替换。

还有一些其他的考虑：

- **逻辑复用而非行为改变：**在使用继承时，应注重复用逻辑而非改变已有行为。子类可以扩展父类的功能，但不应改变父类原有的行为。
- **保持接口一致性**: 子类应遵守父类的契约，包括方法签名、前后置条件、不变式等。
- **使用接口和组合**: 面向接口编程和优先考虑组合而非继承，是实现LSP的有效方法，提供了更大的灵活性和解耦。
- 尽管LSP常与继承关系讨论，**但其核心原则同样适用于接口和实现类之间的关系**。无论是继承还是实现接口，都应确保新的实现或子类不违反基类或接口的预期行为。
- **面向接口编程不仅是实现开闭原则的有效方式**，**也是确保软件组件替换性的关键策略**，与LSP的目标紧密相关。