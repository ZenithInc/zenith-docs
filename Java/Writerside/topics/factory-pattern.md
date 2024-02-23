# 使用工厂模式

这篇文档介绍了如何在 SpringBoot 中优雅的使用工厂方法。借助 Spring 的 `@Autowired` 注解将实现类注入到 `Map` 中，然后通过工厂方法返回实例。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/wwbIWIPWY1wuDRQlPVy4.png" alt="A humorous illustration depicting the challenges of software development with changing requirements. The scene shows a programmer seated at a computer" />

## 应用场景 {id="application-scenarios"}

最近我在写一个转盘抽奖活动的游戏，一个活动会有多个关卡。玩家要在满足每个关卡预定的条件之后才能参与抽奖。每个关卡都有不同的抽奖算法。

通过上面的需求描述，我们立马可以想到，关卡预定的条件和抽奖算法随着业务的发展都会发生变化。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/2UdJO3U8vOm2LMf12jTQ.png" alt="An illustration of a colorful and exciting wheel of fortune game, commonly used in sweepstakes or prize drawings. The wheel is divided into vibrant se"/>

比如关卡的条件，今天可能是充值 500 块钱，明天可能是玩 3 局游戏。 再比如抽奖的算法，第一关是根据你充值的数额来决定你的奖品，第二关就可能是根据奖品配置的概率来开奖。

所以，这两个地方在代码中就不能写死，而是要“面向抽象”编程。

## Coding {id="coding"}

接下来，我们就来编码演示如何优雅地实现上面的需求，分为如下三步:

<code-block lang="plantuml">
@startmindmap
+ 1. 创建抽象类
++ 2. 实现抽象类 
+++ 3. 编写工厂类
@endmindmap
</code-block>

### 创建抽象类 {id="create-an-abstract-class"}

下文我们以关卡条件为例（抽奖算法也是同理）来实现编码。首先我们要创建一个 `ICondition` 的接口:
```Java
public interface ICondition {
    boolean check(String userId);
}
```
这个接口就一个方法，根据传入的 `userId` 返回是否满足关卡的条件。

### 实现抽象类 {id="implements"}

接着我们创建两个实现类, 分别是`BetAmountImpl`(根据下注流水判断是否满足条件)、`CashInImpl` (根据用户充值金额判断是否满足条件):
```Java
@Service
public class BetAmountImpl implements ICondition {
    @Override
    public boolean check(String userId) {
        // 省略实现逻辑
    }
}
@Service
public class CashInImpl implements ICondition {
    @Override
    public boolean check(String userId) {
        // 省略实现逻辑
    }
}
```

### 创建工厂类 {id="create-a-factory-class"}

最后，我们可以创建工厂类， 代码如下:
```Java
@Service
public class ConditionFactory {

    private final Map<String, ICondition> conditonMap;

    @Autowired
    public ConditionFactory(Map<String, ICondition> conditonMap) {
        this.conditonMap = conditonMap;
    }

    public ICondition build(PromotionLevelConditionEnum condition) {
        if (!conditonMap.containsKey(condition.getValue())) {
            throw new NotImplementedException("Not implemented ICondition interface!");
        }
        return conditonMap.get(condition.getValue());
    }
}
```
该函数是一个工厂方法，用于根据给定的条件枚举类型构建相应的条件对象。该方法通过接收一个条件枚举类型参数，根据该枚举类型在条件映射中查找对应的条件对象，并返回该对象实例。如果找不到对应的条件对象，则抛出一个未实现异常。

**这种方式的优点在于，当添加新类型的订单时，你只需要创建一个新的实现了 Order 接口的类并将其作为Spring组件进行注解，而无需修改工厂类的代码。这提高了代码的可维护性和可扩展性。**

## 总结 {id="summary"}

这篇文章讲述了如何在 SpringBoot 中优雅地使用工厂模式。文章首先介绍了使用工厂模式的应用场景，以一个转盘抽奖游戏为例，说明了关卡条件和抽奖算法随业务发展的变化，强调了代码中不能写死，而应该“面向抽象”编程。

然后，文章进入编码部分，分为三个步骤：创建抽象类、实现抽象类、创建工厂类。首先创建了 `ICondition` 接口，用于检查用户是否满足关卡条件。接着，创建了两个实现类`BetAmountImpl` 和`CashInImpl`，分别基于下注流水和用户充值金额判断条件是否满足。最后，创建了工厂类`ConditionFactory`，该类包含一个工厂方法用于根据条件枚举类型构建相应的条件对象，提高了代码的可维护性和可扩展性。

文章强调了这种方法的优点在于添加新类型时，只需创建一个新的实现类并作为 Spring 组件进行注解，而无需修改工厂类代码，从而提高了代码的可维护性和可扩展性。