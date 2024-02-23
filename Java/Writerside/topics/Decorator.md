# 装饰器模式

装饰器模式（Decorator Pattern）是一种结构型设计模式，它提供了一种在运行时动态地给对象添加新职责的灵活替代方案，避免了子类继承的局限性。通过
组合而非继承的方式，装饰器模式能够在不影响其他对象的情况下，为单个对象添加或修改行为，实现功能的扩展和叠加，有效提升代码的可复用性和灵活性。

## 装饰器的例子 {id="example"}

在 Java 中，`InputStream` 以及它的众多子类都运用了装饰器模式，比如 `FileInputStream`、`BufferedInputStream`。其他的基本流也是如此，
比如 `FileOutputSteam`。这里以 `InputStream` 为例。
