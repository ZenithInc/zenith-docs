# Repository

上一篇我们已经定义了 `Entity` 类，建立了和数据表之间的映射。这一篇将讲解定义 `Repository` 接口。`Respository` 在 Spring Data JPA 中
扮演了非常关键的角色。它是一种设计模式，其主要作用是封装数据访问和存储的逻辑。`Repository` 作为持久层的一部分，提供了一个更清晰的抽象层来与数据
源进行交互。


## Repository 概述 {id="override"}

首先我们来定义一个 `Repository` 的接口, 以上篇中的 `UserEntity` 来说
```Java
@Entity
@Table(name = "users")
@Getter
@Setter
public class UserEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    private String password;

}
```

我们为它定义一个 `UserEntityRepository` 的接口:
```Java
public interface UserEntityRepository extends JpaRepository<UserEntity, Long> {}
```

## CURL {id="curl"}

`Repository` 创建并持久化一个实体:
```Java
UserEntity user = new UserEntity();
user.setUsername(body.getUsername());
user.setPassword(body.getPassword());
userEntityRepository.save(user);
```



