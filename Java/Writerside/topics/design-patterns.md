# 关于设计模式

设计模式，作为一种久经验证的最佳实践和经验总结，在软件工程领域扮演着至关重要的角色。它并非具体的代码片段或库，而是一种在面向对象软件设计中解决常
见问题的通用可复用方案。设计模式是历经众多项目迭代与实践锤炼后形成的策略描述，用来指导开发者如何在特定上下文中恰当地组织代码结构、定义类间关系以
及管理对象交互。

## 设计模式的分类 {id="category"}

设计模式分为创建型（Creational Patterns）、结构型（Structural Patterns）、行为型（Behavioral Patterns）三类，如下图所示:

```plantuml
@startwbs
+ 设计模式分类
++ 创建型
+++ 单例模式（Singleton）
+++ 工厂方法模式（Factory Method）
+++ 抽象工厂模式（Abstract Factory）
+++ 建造者模式（Builder）
+++ 原型模式（Prototype）
++ 结构型
+++ 适配器模式（Adapter）
+++ 桥接模式（Bridge）
+++ 组合模式（Composite）
+++ 装饰器模式（Decorator）
+++ 门面模式（Facade）
+++ 享元模式（Flyweight）
+++ 代理模式（Proxy）
-- 行为型
--- 责任链模式（Chain of Responsibility)
--- 命令模式（Command)
--- 解释器模式（Interpreter）
--- 迭代器模式（Iterator）
--- 中介者模式（Mediator）
--- 备忘录模式（Memento）
--- 观察者模式（Observer）
--- 状态模式（State）
--- 策略模式（Strategy）
--- 模版方法模式（Template Method）
--- 访问者模式（Visitor）
@endwbs
```