# 反射 

这篇文档描述了反射在 Java 中的应用，反射有时候也被成为反射 API，这也说明了这是 Java 提供了一组 API。

## 什么是反射 {id="what-is-reflection"}

反射机制大概在 20 世纪 70 年代出现在 LISP 语言中，随后在 SmartTalk 发扬光大，允许程序员通过发送消息来操作对象的内部状态。

<note>
LISP（List Processing） 是一种基于符号计算的语言，在 1958 年由 John MacCarthy 提出，是最早的函数式编程语言之一，也是第一种支持元编程和
反射机制的语言。其主要特点就是元编程，程序员可以通过编写代码来生成和修改代码。
</note>

**而 Java 是在 1996 年的版本中引入反射 API 的，是的程序可以在运行时获取和操作类的信息。** 它是一组 API，定义在 `java.lang.reflect` 包中。

此外，很多动态类型的语言中，也引入了反射机制，比如 PHP、Python 等。

<note>
动态语言中实现反射会比静态语言更简便，而且更灵活地处理不同类型的对象。但是 Java 这样的静态类型的语言需要在编译时进行类型检查，所以需要更加谨慎
处理类型转换和异常。此外，静态类型的语言会在编译时对反射进行优化，所以性能更高。
</note>

## 反射API示例 {id="api-examples"}

下面将会对一些常见的场景，给出一些示例，比如说获取类的信息、动态创建对象、调用方法和访问字段、注解的处理、动态代理等。为了测试，我们先编写一个
测试的类:

```Java
public class Cat {
    public String name = "tom";
    public void printName() {
        System.out.println(name);    
    }
}
```

### 获取类的信息 {id="get-class-meta-info"}

获取类的信息如下:
```Java
// 获取类的 Class 对象
Class<?> clazz = Cat.class;
// 获取类的名称
System.out.println("clazz.getName() = " + clazz.getName());
```

获取方法的信息如下:
```Java
for (Method method : clazz.getMethods()) {
    // output：printName、equals、toString、getClass、hashCode、wait、notify、notifyAll
    System.out.println("method.getName() = " + method.getName());
}
```

获取字段信息:
```Java
for (Field field : clazz.getFields()) {
    // output: name
    System.out.println("field.getName() = " + field.getName());
}
```

### 动态创建对象 {id="create-instance"}

动态创建对象，需要先获得反射对象的构造器，然后使用构造器去创建一个新的实例:
```Java
Class<?> clazz = Cat.class;
Constructor<?> constructor = clazz.getConstructor();
Cat cat= (Cat)constructor.newInstance();
cat.printName();    // output: tom
```
在示例中，我们动态地创建了对象，并调用了`printName()` 方法。

### 解析注解 {id="interpret-annotation"}

下面的示例演示了如何解析注解:
```Java
if (method.isAnnotationPresent(MyAnnotation.class)) {
    // 获取注解对象
    MyAnnotation annotation = method.getAnnotation(MyAnnotation.class);
    // 解析注解的属性值
    String value = annotation.value();
}
```


## 反射的性能 {id="reflects-performance"}

在 Python 之禅中有着重提出了“优美胜于丑陋、明确胜于晦涩、简洁胜于复杂”。在编程中，可读性是我们首要考虑的问题，其次才是性能问题。对反射的性能的
考虑也是建立在这个基础上的。

然后要考虑使用的场合，在框架类库开发、动态配置和扩展性的代码中，使用反射是非常有帮助的。而在一般的业务中，需要避免过度设计、过度使用反射的情况，
除了会引入不必要的性能开销之外，还会增加代码的抽象程度，降低代码的可读性。

如果是非常在性能的情况下，可以使用代替方案，比如代码生成、字节码操作或者使用动态代理等。

## 总结 {id="summary"}

通过反射，我们可以在运行时获取和操作类的信息，而不需要在编译时明确知道类的具体细节。这为我们提供了更大的灵活性和动态性。

在示例中，我们展示了如何使用反射API获取类的信息和动态创建对象。通过Class对象，我们可以获取类的名称、方法信息和字段信息。通过Constructor对象，我们可以实例化对象并调用构造函数。

反射在实际编程中有许多应用场景。例如，当我们需要根据用户的输入动态创建对象时，反射提供了一种便捷的方式。它还可以用于处理注解、动态代理、配置文件解析等。

然而，反射也有一些注意事项。由于反射是在运行时进行的，相对于直接调用方法或访问字段，它的性能开销较大。因此，在性能要求高的场景下，应谨慎使用反射，并考虑其他替代方案。

总而言之，反射是一项强大而灵活的技术，可以在编程中提供许多有用的功能。但是，我们应该根据具体情况权衡使用反射的利弊，并在需要时合理应用它。