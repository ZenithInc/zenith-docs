# 面向对象程序设计

这一系列的文章将来讨论“面向对象程序设计”这个话题，对于软件架构来说，这种风行了数十载的编程范式非常重要，也是目前主流的编程范式。同时我们也会结合其他语言来说明面向对象的一些不足。所以，这一系列的文档会使用 Java 作为主要语言，其他语言作为辅助说明。

## 面向抽象编程 {id="oriented-to-abstract-programming"}

面向对象编程，其实可以有很多说法，偏偏是“面向对象”这个说法容易让人将“对象”作为其本质。我觉得其本质是“抽象”，所以应该说是“面向抽象”编程。

下面，我以一个用户支付的场景来说明什么是面向抽象编程。当我们接到一个“支付”的需求的时候，很快会写出如下的代码：
```Java
public class User {
    
    public void pay(RequestBody params) {
         WechatPay payment = new WechatPay();
         payment->pay();
    }
}
```
上面的代码就是“面向对象”编程，我们在 `register` 方法中直接实例化了一个`WechatPay` 的对象。但试想一下，如果不同的用户可能会选择不同的支付方式呢？比如使用支付宝或者银联支付。我们就必须要修改这个代码，重新测试这个代码。

接着我们使用“面向抽象”编程来重构这段代码。首先创建一个接口类（抽象类）`IPay`:
```Java
public interface IPay {
    void pay();
}
```
然后我们的核心代码就可以换成如下的形式，不再“面向具体的对象”编程，而是面向抽象编程：
```Java
public class User {
    
    public void pay(RequestBody params) {
         IPay payment = factory(params->getPayType());
         payment->pay();
    }
    
    private IPay factory(String type) {
        // WechatPay 和 AliPay 都实现 IPay 接口
        if (type == 'WeChat') {
            return new WeChatPay();
        } else if (type == 'AliPay') {
            return new AliPay();
        }
    }
}
```
我们的核心代码 `pay` 中就是面向抽象编程，并没有说明具体的支付渠道是什么。而是通过 `factory` 工厂方法来决定。这样一来，就算我们要支持其他的支付渠道，也并不需要修改 `pay` 方法。

但是，这样一来你可能会有疑问，我们不是把代码写死在了 `factory` 方法中了吗？是的。但是，可以有两种方式来理解这种做法：

* 可以把 `factory` 方法当作是一种“代码的配置”，类似于 Spring 中的 `Configuration` 类。
* 可以将 `factory` 改写成反射，这样就可以不写具体的类了。在 Spring 中可以通过框架注入，而框架实际上也是使用了反射。

可是还有一个疑问，看起来，重写后的代码更加抽象也更加复杂了。但是其可扩展性更好了，符合开闭原则。

**所以，说是“面向对象”编程，其实是“面向接口”编程。说是面向“接口”编程，其实是“面向抽象”编程。说是“面向抽象”编程，其实是“面向变化”编程。**

很多人，觉得用了“对象”或者“接口”或者“工厂”就是面向对象编程，其实不是。很多人以为“java”是面向对象语言，C 语言是面向过程的语言，其实也不是。面向对象和具体的语言无关，很多人用 C 编写面向对象的代码，很多人用 Java 编写着面向过程的代码。