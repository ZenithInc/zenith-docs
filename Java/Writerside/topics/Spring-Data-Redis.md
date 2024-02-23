# Spring Data Redis

这篇文档介绍了在 SpringBoot 中集成 Redis。

## 添加依赖项 {id="dependence"}

使用 Maven 管理项目依赖，编辑 `pom.xml` 加入如下配置:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## 配置 Redis {id="configure"}

接着配置 Redis 的连接信息。编辑 SpringBoot 的配置文件，例如  `application.properties` 或者 `application.yml`，内容如下:
```
spring.data.redis.host=localhost
spring.data.redis.port=6379
```

