# String

在Java编程语言中，String 类是用来操作字符序列的一个关键类。它提供了一系列的方法来处理文本数据。字符串是不可变的；也就是说，一旦创建了字符串，
就不能更改它。在本文中，我们将深入探讨 String 类，介绍它的基本特性、常用方法以及它在实际编程中的应用。

## 不可变性 {id="immutable"}

Java中的字符串是不可变的。这意味着一旦一个 String 对象被创建，它所包含的字符序列是不能被改变的。如果你尝试修改字符串，实际上会创建一个新的字符
串对象，而原来的字符串保持不变。

所以，在早期的 Java 版本中，对字符串进行大量的拼接、裁剪操作会带来性能问题，比如内存大量占用、频繁 GC。

## StringBuffer 和 StringBuilder {id="string-buffer-and-string-builder"}

为了解决这些问题，Java 提供了 `StringBuilder` 和 `StringBuffer` 两个类。它们之前的区别是，`StringBuffer` 是线程安全的，但是因为使用
`Synchroninzed` 所以性能并不好。而 `StringBuilder` 实现和 `StringBuffer` 是一样的，但是去掉了同步，所以线程不安全。

`StringBuffer` 的示例如下:
```Java
StringBuffer buffer = new StringBuffer("Hello");
buffer.append(" World");
System.out.println("buffer = " + buffer);
```

`StringBuilder` 的示例如下:
```Java
StringBuilder buffer = new StringBuilder("Hello");
buffer.append(" World");
System.out.println("buffer = " + buffer);
```

但是，在实际的编码中并不建议使用 `StringBuffer` 和 `StringBuilder`，因为编译器会自行优化将 String 的拼接等操作转换为 `StringBuilder`。

```Java
String a = "Hello";
System.out.println(a + " World");
```

通过 `javap -c Main.class` 命令，我们查看上面代码的字节码，如下图所示，替换成了 `StringBuilder`。

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/yYcNkHLSy3M5uuxBtdsy.jpeg)

## StringBuffer 的实现方式 {id="string-buffer-implements"}

`StringBuffer` 是一个字符数组，默认初始化的时候会分配 16 个字符长度。

在超出了 16 个字符之后会加倍扩容。但是，它并不会缩容，因为在 Java 中对象一旦分配了内存，就不会动态减少，这是为了减少内存重新分配带来的性能开销。

## 总结 {id="summary"}

本文提供了对 Java 编程语言中 String 类的深入探讨，强调了字符串的不可变性和相关类的使用。首先，String 类是处理字符序列的关键类，其特点是不可
变性， 意味着一旦创建，其内容不可更改。

为了应对由于字符串不可变性导致的性能问题，Java提供了 `StringBuilder` 和 `StringBuffer` 两个类。 `StringBuffer` 是线程安全的，但性能较差；
而 `StringBuilder` 是非线程安全的，但性能更佳。然而，在实际编程中，通常不直接使用这两个类，因为编译器会自动将 String 的拼接 转换为
`StringBuilder` 操作。

此外，介绍了 `StringBuffer` 的实现方式，包括其字符数组的初始分配和扩容机制， 但不会进行缩容以减少性能开销。