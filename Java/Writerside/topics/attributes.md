# 注解

本文将介绍 Java 注解的基本概念、语法和用法，以及如何自定义和使用注解。我们还将探讨一些常见的内置注解，以及如何使用元注解来为注解类添加其他注解。无论你是初学者还是有经验的
Java 开发者，通过学习和掌握 Java 注解，你将能够更好地利用注解机制来提高代码的可读性、可维护性和灵活性。

## 什么是注解 {id="what-is-annotation"}

Java 注解（Annotation）是一种元数据（metadata）机制，用于为 Java
代码提供额外的信息和说明。它们可以在源代码中添加特定的注解标记，以便在编译时、运行时或者在开发工具中进行处理。注解可以用于描述类、方法、字段和其他程序元素，以及为它们提供相关的配置、行为或文档。

Java
注解的引入为开发者提供了一种简洁而强大的方式来定义和使用自定义的元数据。它们可以用于各种用途，如代码分析、编译时检查、运行时处理、文档生成等。注解可以通过反射机制在运行时获取和处理，使得开发者可以根据注解的信息来实现一些特定的逻辑或行为。

## 如何定义注解 {id="how-to-define-an-attributes"}

定义一个基本的注解，示例如下:

```Java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
@Inherited
public @interface LoginOrPublicAccess {}
```

在 Java 中，注解是一种特殊的接口类型，使用 `@interface` 来声明。我们看到类的上面还有三个注解，解释如下:

| 注解            | 含义                                                                                                               |
|---------------|------------------------------------------------------------------------------------------------------------------|
| `@Retention`  | 用于指定注解的保留策略，即注解在什么级别保存，并在运行时可用。常用的保留策略包括 RetentionPolicy.SOURCE、RetentionPolicy.CLASS 和 RetentionPolicy.RUNTIME。 |
| `@Target`     | 于指定注解可以应用于哪些元素上，如类、方法、字段等。常见的目标元素包括 ElementType.TYPE、ElementType.METHOD 和 ElementType.FIELD。                     |
| `@Documented` | 用于指定注解是否包含在 Java 文档中                                                                                             |
| `@Inherited`  | 用于指定注解是否可以被继承                                                                                                    |

解析注解可以参考[《反射》](https://java.doc.hacking.icu/reflects.html)一文。
