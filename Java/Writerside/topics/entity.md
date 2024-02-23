# Entity

这篇文档描述了在 JPA 中如何定义实体，包括在实体中使用各类注解，以及我的一些实践经验。

## 什么是实体 {id="what"}

在Java Persistence API (JPA)中，实体是一个轻量级持久域对象。一个实体代表了数据库中已经存在的表的一行数据，表的每一列映射到一个实体的属性上。
主键对应的列被要求存在于每一个实体类中。整个行的数据封装在一个JPA实体类的一个实例对象中。

下面的示例中，定义了一个实体:
```Java
@Entity
@Table(name = "users")
public class UserEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

在 JPA 中，如果没有通过 `@Table` 注解在类上方声明表名，则采用默认的规则映射真实的数据表名。比如我有一张表名为 `users`, 则默认映射的实体名
为 `UsersEntity`。

> 我觉得这并不是一种优雅的约定。因为我在给数据表命名的时候，倾向于使用复数形式。比如 `users`。因为一张数据表中存储的是多个 `user` 信息。但是从
实体角度来说，一个实体表示的就是一个 `user`, 而不是多个。所以实体的命名应该使用单数形式。所以，不能简单的采用 `users` 表映射为
`UsersEntity`，在语意上我觉得并不妥当。这也是在相当多的团队项目中，采用的一种合理的做法。

接着，我们看到 `@Id` 注解，它表示 `id` 字段是数据库中的主键。而 `@GeneratedValue` 注解，表示 `id` 字段值的生成策略，有如下选项:

| 策略                        | 描述                                                    |
|---------------------------|-------------------------------------------------------|
| `GenerationType.AUTO`     | 默认的生成策略，根据底层的数据库自动选择主键生成机制                            |
| `GenerationType.IDENTITY` | 主键由数据库自动进行增长，主要配合数据库的自增长列，比如 MySQL 的 `auto_increment` |
| `GenerationType.SEQUENCE` | 通过序列生成主键，要求数据库支持序列。对应于数据中的 `sequence` 对象              |
| `GenerationType.TABLE`    | 使用一个特定的数据库表进行主键值的维护                                   |

## 定义属性 {id="properties"}

在实体上定义属性，建立和数据表字段的映射。比如下面这个实体，定义了一个简单的 `UserEntity`：
```Java
@Entity
@Table(name = "users")
public class UserEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    private String password;

    private String status;

    private Long latestLoginTime;

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;
}
```

默认情况下，使用驼峰形式来映射数据表中的字段。如果想要自定义可以使用 `@Column` 注解，这个注解的参数如下表所示:

| 参数               | 解释                       | 示例                                                           |
|------------------|--------------------------|--------------------------------------------------------------|
| name             | 数据库列的名称。默认为实体属性的名称。      | `@Column(name = "first_name")`                               |
| nullable         | 指示列是否允许为空。默认为 true。      | `@Column(nullable = false)`                                  |
| unique           | 指示列是否需要唯一性约束。默认为 false。  | `@Column(unique = true)`                                     |
| length           | 列的最大长度。通常用于字符串列。默认为 255。 | `@Column(length = 100)`                                      |
| precision        | 列的精度（总位数）。通常用于数值列。默认为 0。 | `@Column(precision = 10)`                                    |
| scale            | 列的小数位数。通常用于数值列。默认为 0。    | `@Column(scale = 2)`                                         |
| insertable       | 指示是否允许插入数据到列。默认为 true。   | `@Column(insertable = false)`                                |
| updatable        | 指示是否允许更新列的值。默认为 true。    | `@Column(updatable = false)`                                 |
| columnDefinition | 定义数据库列的自定义 SQL 类型和约束。    | `@Column(columnDefinition = "VARCHAR(100) NOT NULL UNIQUE")` |

## 定义属性为枚举类型 {id="enum"}

比如上面的 `UserEntity` 中的 `status` 字段，定义为 `String` 类型，并不是一种提倡的做法。在实际的生产中，我们会将它定义为枚举类型，一方面可
以防止字面量写错，另一方面可以随时更新枚举的值，而不用到处修改字面量。

首先，我们定义一个 `UserStatusEnum` 的类，代码如下所示:
```Java
@Getter
@AllArgsConstructor
public enum UserStatusEnum {

    VALID("VALID"),

    INVALID("INVALID");

    private final String name;
}
```

接着，在 `UserEntity` 实体中，更改字段的类型为 `UserStatusEnum` 并且添加 `@Enumerated` 注解，代码如下:
```Java
@Entity
@Table(name = "users")
public class UserEntity {

    // 省略其他代码
    
    @Enumerated(EnumType.STRING)
    private UserStatusEnum status;
}
```

`@Enumerated` 表示这个字段值是枚举，有两个可选项：

* **`EnumType.STRING`** : 则使用枚举的名称(`name()`) 返回字符串并存储到数据库，建议使用

* **`EnumType.ORDINAL`** : 它使用枚举的序数（枚举声明中的位置，从 0 开始）来存储到数据库, 不建议使用

## 属性自动写入更新 {id="auto-update"}

在上面定义的实体中，有两个属性，分别是 `createdAt` 和 `updatedAt`。在数据表中，它们是两个 `bigint` 类型的数据，用来存储时间戳。上面的示例
中的使用 `Long` 类型声明属性，也不是一种好的做法。我们可以在写入数据库的时候，自动写入时间戳，读到程序中后自动转为 `LocalDateTime` 类型。

```Java
@Entity
@Table(name = "users")
public class UserEntity {

    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime createdAt;

    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() {
        LocalDateTime now = LocalDateTime.now();
        this.createdAt = now;
        this.updatedAt = now;
    }

    @PreUpdate
    protected void onUpdate() {
        this.updatedAt = LocalDateTime.now();
    }

}
```

示例代码中，出现了三个新的面孔，分别是 `@Temporal` 、`@PrePersist` 以及 `@PreUpdate` 三个注解。我们分别解释它们:

- `@PrePersist` 注解的 `onCreate` 方法在实体首次被持久化（即保存到数据库）前被调用。它设置了 `createdAt` 和 `updatedAt` 字段的值为当前时间。
- `@PreUpdate` 注解的 `onUpdate` 方法在实体更新时被调用。它更新了 `updatedAt` 字段的值为当前时间。
- `@Temporal` 注解用于映射 `LocalDateTime` 等日期时间的类型，有三个选项，如下:

| 选项                       | 描述                          |
|--------------------------|-----------------------------|
| `TemporalType.DATE`      | 仅映射日期部分（年月日），不包括时间          |
| `TemporalType.TIME`      | 仅映射时间部分（小时、分钟、秒），不包括日期      |
| `TemporalType.TIMESTAMP` | 映射完整的日期和时间（年月日小时分钟秒，通常包括毫秒） |
