# 六大原则之开闭原则

开闭原则（Open-Close Principle，OCP）是伯兰特 · 迈耶（Bertrand Meyer）在 1988 年发表的《面向对象软件构造》中提出的。

## 什么是开闭原则 {id="what-is-ocp"}

Bertrand Meyer 在《面向对象构造》一书中提出了开闭原则，其原文如下：

> Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.


软件的实体（不限于类、模块、函数、服务、系统）在设计中应该对扩展是开放的，对修改是封闭的。**开闭原则的核心其实就是面向抽象编程**，而面向对象中，抽象指的是接口（包括抽象类），所以面向接口编程。

举例说明，当我们要开发一个支付模块的时候，一般需要考虑接入多个不同的支付渠道，比如支付宝、微信。这时候，违反开闭原则的设计如下:

```java
public class Pay {
    public void request(String channel) {
        if (channel.equals("ALIPAY")) {
            // call the alipay sdk api
        } else if (channel.equals("WEIXIN")) {
            // call the weixin adk api
        }
    }
}
```

上面的代码存在如下两个问题：

- 当引入了新的支付方式的时候，就必须要修改 `request`这个方法，要知道支付涉及金钱，在系统中是非常核心的，频繁修改这样核心的方法是不能被接受的。

- `request` 方法随着支付渠道越来越多，会变得越来越庞大和复杂，越来越难以维护。

为了符合开闭原则，我们采用拆分为接口的方式来重构上面这个方法:

```java
public interface IPay {
    void request();
}

public class WeixinPay implements IPay {
    @Override
    public void request() {
        // call the weixin pay sdk api
    }
}

public class Alipay implements IPay {
    @Override
    public void request() {
        // call the alipay sdk api
    }
}
```

然后 `pay`方法重构如下:

```java
public class Pay {

    public void request(String channel) {
        factory(channel).request();
    }

    public IPay factory(String channel) {
        if (channel.equals("ALIPAY")) {
             return new Alipay();
        }
        return new WeixinPay();
    }
}
```

通过重构，我们的代码符合了开闭原则，实现了对扩展（接口实现）是开放的，对修改是封闭的（没有理由去修改 `request()`方法，这个核心的方法是稳定的），从而提升了程序的可扩展性。

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/oU1DVXC3ZChZwixSO4iX.png)

有如下好处：

- **扩展性强**：引入新的支付渠道时，只需添加新的 `IPay` 实现，无需修改现有的 `Pay`类或其他支付渠道的实现，符合开闭原则。

- **低耦合**：通过将不同的支付渠道实现分离到不同的类中，降低了类之间的耦合度，使得每个类更加专注于其职责。

- **易于维护**：每个支付渠道的实现细节被封装在各自的类中，便于理解和维护。

## 使用继承实现开闭原则 {id="implementing-opc-though-extends"}

在多数场景下，其实使用接口是更优的做法，但是也可以使用继承来实现开闭原则。但是在某些特殊的场景下，你需要保留一些通用的实现的时候，使用继承来实现开闭原则会更好。

比如说，我要实现一个图形编辑器，在幕布中可以绘制不同的图形，但是图形的绘制方法都不尽相同:

```java
public abstract class Shape {
    protected int x, y;
    protected String color;

    public Shape(int x, int y, String color) {
        this.x = x;
        this.y = y;
        this.color = color;
    }

    public abstract void draw();

    public void move(int newX, int newY) {
        this.x = newX;
        this.y = newY;
        System.out.println("Move shape to: " + x + ", " + y);
    }
}

```

然后编写子类:

```java
public class Circle extends Shape {
    private int radius;

    public Circle(int x, int y, String color, int radius) {
        super(x, y, color);
        this.radius = radius;
    }

    @Override
    public void draw() {
        System.out.println("Drawing circle at: (" + x + ", " + y + ") with radius " + radius + " and color " + color);
    }
}
```

在来编写更多的子类:

```java
public class Rectangle extends Shape {
    private int width, height;

    public Rectangle(int x, int y, String color, int width, int height) {
        super(x, y, color);
        this.width = width;
        this.height = height;
    }

    @Override
    public void draw() {
        System.out.println("Drawing rectangle at: (" + x + ", " + y + ") with width " + width + " and height " + height + " and color " + color);
    }
}
```

所以，当不需要复用代码的时候，优先使用接口的方式实现 OCP，但是当需要代码复用的时候可以优先使用抽象类。

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/z2DbnFPb49EQ2TqOND1Y.png)

就像上面的代码所示，除了逻辑的复用外，还有包含有状态（比如 `x`、`y`、`color`、`height`，使用接口来实现也不是不可以，但是如果存在大量的状态和行为的复用的时候，使用继承会更好。

> 有可以优先使用接口，当出现行为和状态复用的时候，再采用继承的方式重构。

## 总结 {id="summary"}

在本文中，我们深入探讨了开闭原则（Open-Closed Principle, OCP），它是面向对象设计原则中的一个核心概念，强调软件实体（如类、模块和函数）应该对扩展开放，对修改封闭。这意味着在不改变现有代码的前提下，我们可以通过扩展来增加新功能，从而提高软件系统的可维护性、可扩展性和稳定性。

- **开闭原则的定义**：软件实体应该易于扩展，但是在扩展其功能时，不应该修改原有的代码。

- **实现方法**：我们讨论了实现开闭原则的两种主要方法：通过接口和通过继承。接口提供了一种灵活的方式来定义可替换的行为，而继承允许共享基础行为并通过派生类进行扩展。

- **接口与继承的比较**：我们讨论了在不同情境下选择接口和继承的考量，指出接口提供了更高的灵活性和低耦合度，而继承则在共享行为和状态管理方面更有优势。

开闭原则是构建可扩展、可维护软件的关键。通过遵循这一原则，开发者可以更容易地适应需求变化，减少因扩展功能而对现有系统造成破坏的风险。不过，正确地应用开闭原则需要根据具体情况权衡使用接口还是继承，以及如何设计系统的架构以促进扩展性和维护性。