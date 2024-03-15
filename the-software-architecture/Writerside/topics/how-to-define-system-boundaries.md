# 如何确定系统边界 

这篇文档讨论了如何确定系统边界这个话题，在软件架构中，这是非常基础也非常重要的一个话题。换句话说，好的软件设计的前提是有良好的边界划分。

<note>
这篇文章中，我们所谓的系统边界中的系统一词，在不同的语言、场景下有着不同的表述。比如根据不同的粒度可以描述为系统、模块、接口（API）、类、接口、方法、签名等。
</note>

## 重要的原则 {id="key-principles"}

在确定系统边界的设计中，需要遵循两个重要的原则：**职责单一原则（SRP: Single Responsibility Principle）**和**松耦合（Loose Coupling）**。

* **职责单一**：引起系统（模块）发生变化的原因只能有一个。举例来说，比如在一个 `UserManager` 的类中，有一个 `sendSms` 的方法，就违反了这个原则。如果发送短信的逻辑变了，这个用户管理类也需要发生变化。
* **松耦合**: 违反了职责单一的同时，其实也不符合松耦合这个原则。单一的组件承担了太多的职责，不同的组件之间紧密关联，常常牵一发动全身。

## DSSA 方法 {id="domain-storytelling-software-architecture"}

![Domain Storytelling Software Architecture](http://file-linker.oss-cn-hangzhou.aliyuncs.com/45CTD7mpsLg5NqfiQudN.png)


## 相关文章 {id="related-articles"}

1. [Good software architectures are mostly about boundaries](https://federicoterzi.com/blog/good-software-architectures-are-mostly-about-boundaries)