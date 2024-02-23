# Lombok

Lombok 可以帮我们简化一些重复代码的编写，比如说 getter, setter 之类的。这篇文档描述了它的相关使用。

### 安装

使用 Maven 安装，如下:

```xml
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<version>1.18.24</version>
	<scope>provided</scope>
</dependency>
```

### 简化 Getter 和 Setter

在 Java 中，我们需要编写大量的 Getter 和 Setter 方法。我们可以使用 Lombok 来简化这类枯燥的代码的编写，如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/3GarSmUhLEJRNK7m3vP1.png" alt="getter_and_setter"/>

使用 `@Data` 注解也可以实现同样的效果，但是这个注解会额外生成其他的代码，一般不建议使用。

### 简化构造函数

在 Lombok 中提供了一些关于简化构造函数编写的注解，我们分别来介绍一下:

- `@NotNull`： 可以用来给修饰成员变量，表示这个成员变量不能为空
- `@AllArgsConstructor`: 用来修饰类，生成包含所有参数的构造函数
- `@NoArgsConstructor`: 用来修饰类，生成无参的构造函数
- `@RequiredArgsConstructor`: 生成包含所有使用`@NotNull` 注解修饰的成员变量的构造函数

具体的使用如下图所示:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/uxnMhkEvljzvKmYLzmBi.png" alt="constructor"/>

### 简化 Builder 模式编写

如果我们的类中有很多的参数，我们又希望根据自己指定的若干参数来构造实例，就可以使用 Builder 模式来编写代码。而在 Lombok 中提供了对应的注解如下代码所示:

```java
@Builder
public class PersonDTO {
    String name;
    Integer age;
}
```

这样一来，我们就可以通过下面的方式来构造这个实例了:

```java
PersonDTO person = PersonDTO.builder()
	.name("Bob")
	.age(19)
	.build();
```

但是需要注意，默认情况下，`PersonDTO` 这个类是只能通过这种 Builder 模式创建实例的。因为 `@Builder` 注解为这个类生成了一个私有的无参的构造方法，所以它不再能够被实例化。