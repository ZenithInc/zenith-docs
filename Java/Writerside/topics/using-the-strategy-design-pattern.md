# 使用策略模式

在面向对象编程中，策略模式是一种设计模式，允许在运行时选择算法的行为。这篇文档通过营销活动的这个场景来解释如何应用策略模式。

场景是做一个营销活动的系统，不同的用户匹配不同的活动。很快，我们就可以写出如下代码:

```Java
public void getPromotion(User user) {
    if (user.getType() == "A") {
       return '活动A';
    } else if (user.getType() == "B") {
        return '活动B';
    } else if (user.getType() == "C") {
        return '活动C';
    }
}
```

但是这样的代码非常直白，也非常不容易扩展，真实场景判断的条件是非常复杂的，这里的代码就会写的非常冗长。而且如果我要增加活动互斥、临时取消某个用户类型的匹配，都需要对整个方法进行完整的测试。

接着，我们使用策略模式进行重构。首先，我们编写一个策略类：

```Java
public interface IPromotionStrategy {

    /**
     * 是否启用策略
     */
    boolean isEnabled();

    /**
     * 返回是否符合策略
     */
    boolean isEligible(User user);

    /**
     * 获取符合策略的活动列表
     */
    String getList(User user);

    /**
     * 获取策略权重
     */
    Integer getWeight();
}
```

然后我们实现一个策略:
```Java
public class UserAImpl implements IPromotionStrategy {

    @Override
    public boolean isEnabled() {
        // 表示策略是否启用
        return true;
    }

    @Override
    public boolean isEligible(User user) {
        // 是否满足策略
        return user.getType == "A";
    }

    @Override
    public String getList(GetPromotionUserParams user, List<PromotionDO> list) {
        // 返回满足策略后返回的活动
        return '活动A';
    }

    @Override
    public Integer getWeight() {
        return 999;
    }
}
```
完成了上面这些抽象和实现之后，我们就可以面向抽象编程了:
```Java
public class StrategyExecutor {

    public List<PromotionDO> getEligiblePromotion(User user) {
        List<String> result = new ArrayList<>();
        // 遍历策略
        for (IPromotionStrategy strategy : getStrategies()) {
            // 如果匹配策略，则返回匹配的活动
            if (strategy.isEligible(user, eligibleList)) {
                strategy.getList(user, eligibleList);
            }
        }
        return result;
    }
    
    private List<IPromotionStrategy> getStrategies() {
        List<IPromotionStrategy> list = new ArrayList();
        list.add(new UserAImpl());
        return list;
    }
} 
```

最后我们的客户端的代码就非常简单了:
```Java
public void getPromotion(User user) {
    (new StrategyExecutor())->getEligiblePromotion(user);
}
```