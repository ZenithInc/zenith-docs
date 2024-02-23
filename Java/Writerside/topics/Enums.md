# 枚举

从 Java5 开始提供了枚举的特性，枚举实际上是一类常量，表现形式是类，其存储的是一组常量。枚举改变了在代码中直接使用字面量或者属性值来表示常量的做法，为常量提供了统一的编写形式。

## 基本使用 {id="usage"}

在没有枚举类之前，我们定义常量如下形式:
```Java
public class TestWeek {
    public static final int MONDAY = 1;
    public static final int TUESDAY = 2;
    public static final int WEDNESDAY = 3;
    public static final int THURSDAY = 4;
    public static final int FRIDAY = 5;
    public static final int SATURDAY = 6;
    public static final int SUNDAY = 7;

    @Test
    public void testWeek() {
        System.out.println(TestWeek.MONDAY);
    }
}
```
当我们使用了枚举类之后，同样的功能，写法就简单了很多:
```Java
public enum TestWeekEnum {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

在很多时候，我们希望枚举的值并不是无意义的数值序号，而是有意义的单词:
```Java
public enum TestWeekEnum {
    MONDAY("MONDAY"), 
    TUESDAY("TUESDAY"), 
    WEDNESDAY("WEDNESDAY"), 
    THURSDAY("THURSDAY"), 
    FRIDAY("FRIDAY"), 
    SATURDAY("SATURDAY"), 
    SUNDAY("SUNDAY"),
    ;
    
    private final String value;
    
    TestWeekEnum(String value) {
        this.value = value;
    }
    
    public String value() {
        return this.value;
    }
}
```

在 Java 中，枚举的语法是有一些奇怪的，它是一个受到限制的类。比如我们可以定义一个构造方法，但是可见性修饰必须是 `private`。如果定义了构造方法，那么枚举值的定义也必须加入同样的参数，比如构造方法中的形参是 `value`, 那么枚举值就必须写成 `MONDAY("MONDAY")`这样的形式。所以，这个构造方法，包括属性，都是围绕着枚举值转的。

方法也一样，我们定义了一个名为 `value()` 的方法，那么每一个枚举值都会有这个 `value()` 的方法。

本质上，枚举类和其他的类并没有区别。在编译之后也会转换为正常形式的类，并且继承 `java.lang.Enum` 类。

## 获取所有的枚举值 {id="get_enum_names"}

有时候，我们需要获取枚举的所有的值。你可以通过 `values` 的方法:
```Java
TestWeekEnum.values();
```
有时候，你并不知道自己要操作的是什么类型的枚举，这时候可以借助泛型来获取任意类型的枚举的所有的属性:
```Java
public static <T extends Enum<T>> boolean inEnum(String value, Class<T> enumType) {
    List<String> all = EnumSet.allOf(enumType).stream().map(Enum::name)
        .collect(Collectors.toList());
    return all.contains(value);
}
```
上面的方法判断了给定的`value` 是否在 `enumType` 中包含。