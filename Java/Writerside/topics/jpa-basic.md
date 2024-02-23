# 快速入门

JPA（**J**ava **P**ersistence **A**PI），可以说是一个 ORM 框架，支持众多的关系性或非关系性数据库。这篇文档介绍了 JPA 的大部分用法，通过学习这些用法能较好掌握 JPA。

## JPA OR MyBatis {id="jpa-or-mybatis"}

从市场来说，目前在国内使用 MyBatis 的比使用 JPA 的多，但是新的项目逐渐开始使用 JPA。从国内外来说，在国内使用 MyBatis 多，在 JPA 使用的也很多。

那么究竟使用哪个呢？两个多要会的基础上，在公司里用哪个就使用哪个，自己的项目喜欢哪个就使用哪个。

## 安装依赖 {id="install"}

总共要安装三个依赖，Maven 依赖的配置如下：
```xml
<dependencies>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
</dependencies>
```

## 数据库配置 {id="configure"}

配置分为两部分，一部分是 MySQL 的配置:
```yaml
spring:
	datasource:
		url: jdbc:mysql://localhost:3306/missyou?characterEncoding=utf-8&serverTimezone=GMT%2B8
		username: root
		password: .PBY_yp8HTpuRWEZb2*WHjaoF@GN_kAe
```
另一部分是关于 JPA 的配置:
```yaml
spring:
	jpa:
	  hibernate:
	    ddl-auto: update
```
`ddl-auto` 配置项表示是否自动创建 DDL，`update` 表示只执行更新的部分，还有一个选项是 `create-drop` 这个一般在测试的时候使用，生产环境下
禁用，会导致所有数据被删除。**通常，这个可以配置称 `none` 或者不配置，特别是在生产环境中** 。

如果要在控制台输出 SQL 语句的话，可以在 `application-dev` 中加入如下配置:
```yaml
spring:
  jpa:
    properties:
      hibernate:
        show_sql: true
        format_sql: true
```
这样打印出来的 SQL 是不包含参数的，如果希望参数也打印出来，可以添加如下配置:
```yaml
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=trace
```
## 创建实体类 {id="entry"}

我们以电商中的 `Banner` 为例，创建了一个实体类:
```java
@Entity
@Table(name = "banner")
public class Banner {

    @Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(length = 64)
    private String name;
    
    @Transient
    private String description;
    
    private String img;
    
    private String title;
}
```
主要是一些注解，我们来解释一下:

- `@Entity` 标识一个实体类，我们的 JPA 就会根据这个标志来自动创建  DDL
- `@Table` 设定表名，在我们实体类和真实的表名不一致的时候使用
- `@Id` 表示实体的主键
- `@Column` 设置一些字段的属性，比如上面 `length` 对应 MySQL 中的 `varchar` 类型
- `@Transient` 表示这个字段在数据表中不予以创建
- `@GeneratedValue` 表示这个字段值的生成方式，使用 `GenerationType.IDENTITY` 这个常量表示这个值使用数据库提供的自增长生成
- `@Enumerated` 表示这个字段值是枚举，如果设定为 `EnumType.STRING` 则使用枚举的名称(`name()`) 返回字符串并存储到数据库，而不建议使用
`EnumType.ORDINAL` 它使用枚举的序数（枚举声明中的位置，从 0 开始）来存储到数据库。



在关系型数据中，存在三种关系，分别是一对一、一对多、多对多。其中一对一通常是因为在一个表中字段太多了，所以会创建主表和从表，这样才延伸出一对多的
关系。而多对多通常需要创建三张表，其中一张为关系表，记录两者之间的关联。业务复杂的时候还会验证出一些关系字段。

那么在 JPA 中，是如何表示这些关系的呢？

## 一对多



上面我们已经创建了一个名为 `Banner` 的实体，一个 `Banner` 可能会对应多个 `BannerItem`。所以下面我们创建一个名为 `BannerItem` 的关系实体：
```java
@Entity
public class BannerItem {
    @Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String img;
    private String keyword;
    private Short type;
    private String name;
}
```
然后在之前的 `Banner` 实体类中添加一个属性:
```java
@Entity
public class Banner {
	// ...其他字段省略...
    @OneToMany
    private List<BannerItem> items;
}
```
我们通过 `@OneToMany` 来修饰这个字段，建立`Banner.items` 和 `BannerItems` 表之间的关系为一对多。

自动创建数据库后，我们看到总共生成了三张表，分别是 `banner` 、`banner_item` 以及 `banner_items` 表。我们不是说多对多才需要创建三张表的吗？为什么一对多也创建了三张表呢？

这是因为我们在上面实体中并没有指定关联的外键，所以 JPA 就自己创建了第三张表来记录两个实体之间的关系。但我们并不需要这么做，所以只需要指																																																								定外键即可:
```java
@Entity
public class Banner {
	// ...其他字段省略...
    @OneToMany
    @JoinColumn(name = "bannerId")
    private List<BannerItem> items;
}
```

## 一对多的双向配置 {id="one-to-many-two-way-configure"}

**在两个关联的实体中的任意一个实体配置关系，就称之为单项配置，而在两个实体中同时配置关系，就称为双向配置。**举例说，上面我们示例中是一对多的单项配置。我们试想在业务中，有时候我们是通过 `Banner` 来查询 `BannerItem` ,正如上面的这种单项配置，既在 `Banner` 实体类中配置和 `BannerItem` 的关系来实现关联查询。但有时候，我们也会希望同时通过 `BannerItem` 来查询对应的 `Banner` 记录，这时候就需要通过在 `BannerItem` 中配置和 `Banner` 的关系实现关联查询。

我们在 `BannerItem` 中新增一个属性来表示它和 `Banner` 之间的关系:
```java
@Entity
public class BannerItem {
	// ...省略其他字段...
    @ManyToOne
    private Banner banner;
}
```

在双向的配置中，有两个概念是**关系维护端**和**关系被维护端**。在一对多的配置中，关系维护端指的是多的端，比如说一个 Banner 对应着多个 BannerItem, 那么 BannerItem 就是关系维护端。而关系被维护端指的是一端，即 Banner 就是一端，就是关系被维护端。

然后我们来说如何配置双向关系，在单项配置中，我们使用 `@JoinColumn` 来表示关联字段。那么在双向的配置中，这个注解写在哪里呢？是 `Banner` 还是 `BannerItem` 呢？应该写在 `BannerItem` 这个关系维护端中。于是，我们需要在 `Banner` 类中删除 `@JoinColumn` 的注解，加入到 `BannerItem` 类中去:
```java
@Entity
public class BannerItem {
	// ...省略其他字段...
	// 在双向配置中，bannerId 是会自动生成的，不能显示配置，所以下面这行也需要删掉
	// private Long bannerId;
    @ManyToOne
	// 如果你希望保留上面的 bannerId 成员
	// 下面这个就需要改写成 
	// @JoinColumn(insertable = false, updatable = false, name = "BannerId")
	@JoinColumn(name = "bannerId")
    private Banner banner;
}
```
最后需要在 `Banner` 类的`items` 属性的 `OneToMany` 注解中加入 `mappedBy` 属性, 它的值就是 `BannerItem` 类中的 `banner` 变量名:
```java
@Entity
public class Banner {
	// ...省略其他字段...
    @OneToMany(mappedBy = "banner")
	// 下面这行就需要删掉
    // @JoinColumn(name = "bannerId")
    private List<BannerItem> items;
}

```

## 多对多 {id="many-to-many"}

前面的 `Banner` 以及 `BannerItem` 已经不足以我们演示多对多的配置了。所以我们创建两个其他的实体，一个名为 `Spu` ：
```java
@Entity
public class Spu {
    @Id
    private Long id;
    @ManyToMany
    private List<Theme> themes;
}
```
接着我们来创建一个名为 `Theme` 的实体类:
```java
@Entity
public class Theme {
    @Id
    public Long id;
}
```
然后重启应用，观察生成了表结构。一共生成了三张表，分别为 `spu` 、`theme` 以及它们之间的关联表 `spu_themes`。

## 多对多的双向配置 {id="many-to-many-two-way-configure"}

和一对多的双向配置是一样的道理，多对多也可以设置双向配置，也分为关系维护端和关系被维护端。不一样的是，在一对多的时候，关系维护端必须是多端，而多对多的时候，关系维护端是任意一个实体类都可以。

针对 `Spu` 和 `Theme` 的双向配置如下, 在上文的基础上，更改 `Theme` 加上 `@ManyToMany` 的注解:
```java
@Entity
public class Theme {
    @Id
	@ManyToMany
    public Long id;
}
```
重新生成表结构，发生和预期是不一样的，除了 `spu` 以及 `theme`, 还是生成了 `spu_themes`和 `theme_spu_list` 表，一共是四张表。这是为什么呢？其实和之前一对多生成三张表是一样的道理，没办法确定关系维护端和关系被维护端。所以，我们需要通过 `@ManyToMany` 注解中的 `mappedBy` 属性来指定:
```java
@Entity
public class Spu {
    @Id
    private Long id;
    @ManyToMany(mappedBy = "spuList")
    private List<Theme> themes;
}

```
重启应用重新生成表结构后，生成的就是三张表了。但是我们发现在生成的 `theme_spu_list` 表中的 `spu_list_id` 字段的命名并不如预期，我们希望是 `spu_id`。这时候，我们可以在被关联的实体中加上`@JoinTable`:
```java
@Entity
public class Theme {
	// ...其他字段省略...
    @ManyToMany
    @JoinTable(name = "theme_spu_list", joinColumns = @JoinColumn(name = "theme_id"),
            inverseJoinColumns = @JoinColumn(name = "spu_id"))
    public List<Spu> spuList;
}
```
重新生成表结构即可。

## 执行查询语句 {id="exec-sql"}

在 JPA 中，执行查询语句一共有 8 中方式。所以，它具有很强大的查询能力。先来介绍第一种查询方式，采用接口继承 `JpaRepository` 的方式。我们首先创建一个 `BannerRepository` 接口:
```java
@Repository
public interface BannerRepository extends JpaRepository<Banner, Long> {
    Optional<Banner> findByName(String name);
}
```
需要注意的是 `JpaRepository` 是一个泛型类，接受两个参数，第一个 `Banner` 指的是返回的实体类，第二个 `Long` 指的是主键 ID 的类型。

然后在这个接口中我们定义了一个方法的声明，需要注意的是方法名是固定的写法 `findByName` 根据 Banner 的名称来获取一条 Banner 的记录。根据这个方法的名称， JPA 可以自行推断出我们的查询意图，为我们自动构建查询 SQL。下面的代码就可以查询出数据表中结果:
```java
this.bannerRepository.findByName(name);
```
除此之外，你还可以使用一些内置的方法，比如说 `findAll` 之类的，可以在方法列表查阅。
> 注意：遇到了一个坑，数据表中的数据如果主键是空的，则查不出来......


## 懒加载和急加载 {id="lazy-loading-and-emergent-loading"}

当我们执行上文中提到的 `this.bannerRepository.findByName(name)` 这行代码的时候，查询出来的只是 Banner 表中对应的记录，并不会查询出 `BannerItem` 的关联记录。这是因为默认情况下，JPA 采用的是懒加载，即如果你不使用到 BannerItem, 则不会查询它，以此来节省查询的开销。

与之对应的是急加载，我们可以通过注解来告诉 JPA 采用这种方式:
```java
@Entity
public class Banner {
	// ...省略其他字段...
    @OneToMany(fetch = FetchType.EAGER)
    @JoinColumn(name = "bannerId")
    private List<BannerItem> items;
}
```
通过给 `@OneToMany` 注解设置 `fetch`属性的值为 `FetchType.EAGER` 来设定采用急加载。懒加载和急加载我们可以通过上文介绍了在控制台输出 SQL 的方式来观察。

## 禁止 JPA 生成外键约束 {id="disable-foreign-key"}

标准的做法是在每一个需要的实体的上修改如下注解:
```java
@JoinTable(foreignKey = @ForeignKey(value = ConstraintMode.NO_CONSTRAINT))
public List<Spu> spuList;
```
不过在我当前的版本尝试了一下，这种做法是没有用的。另外，可以通过废弃的 `@foreignKey` 来实现:
```java
@org.hibernate.annotations.ForeignKey(name = "null")
public List<Spu> spuList;
```
但是 `@org.hibernate.annotations.ForeignKey` 这个已被标记为 `@deprecated` ，可能废弃不应该依赖。

另外，当前也没有全局配置的做法来关闭这种生成外键约束的行为，希望之后的版本可以有。

## JPA 序列化 {id="serialization"}

默认情况下， JPA 采用了 `jackson` 序列化包，有的项目中会将其换成阿里的 `fastjson`。这里要说的是，替换掉它对于项目来说并没有太多实际的价值，速度的提升对项目并不会有太多的影响。但是 `fastjson` 提供的接口会比 `jackson` 要好用一些。

下面的配置可以让 JPA 将时间转换为时间戳的格式，另外返回客户端的字段全部采用蛇形命名，比如 `createTime` 改为 `create_time`：
```yaml
spring:
  jackson:
    serialization:
      write-dates-as-timestamps: true
    property-naming-strategy: SNAKE_CASE
```
如果有的字段不希望被序列化后返回给前端，可以使用 `@JsonIgnore`注解。

另外，序列化的操作会触发 JPA 将那些导航字段也一并查询出来，即使你设定了默认的懒加载。我们可以通过 `VO(View Object)` 对象来优化这一问题。

## 总结 {id="summary"}

这篇文档全面介绍了 JPA（Java Persistence API），一个支持多种关系型和非关系型数据库的 ORM 框架。文档首先比较了 JPA 和 MyBatis 在国内外的使用情况，建议根据个人或公司的偏好选择合适的框架。接着，文档详细介绍了 JPA 的设置步骤，包括安装依赖、数据库配置、创建实体类等。在实体类创建部分，文档详细解释了 JPA 注解的用法，如 `@Entity`、`@Table`、`@Id` 等，以及如何表示不同的数据关系（一对一、一对多、多对多）。

文档还探讨了 JPA 中的双向配置，详细解释了如何在关联实体间建立双向关系，并强调了关系维护端和被维护端的概念。此外，文档介绍了 JPA 的查询能力，包括使用 `JpaRepository` 接口执行查询和理解懒加载与急加载的区别。对于 JPA 生成外键约束的禁用方法也进行了讨论，虽然提到了一些特定注解，但指出这些方法在当前版本可能无效。

最后，文档讨论了 JPA 序列化的默认设置和自定义配置，比如使用 `jackson` 或 `fastjson`，以及如何调整时间格式和命名策略。文档还提到，序列化操作可能触发懒加载字段的查询，建议通过使用 VO（View Object）对象进行优化。