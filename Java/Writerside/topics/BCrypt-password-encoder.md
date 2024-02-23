# BCryptPasswordEncoder

在很多的项目中，只是简单实用 MD5 散列算法加密用户密码。这篇文档介绍了使用 Spring Security 中提供的 BCryptPasswordEncoder 对密码进行加密校验，这是密码加密的最佳实践。

## Install

Maven 引入依赖:
```xml
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-crypto</artifactId>
  <version>5.7.5</version>
</dependency>
```

## Usage

封装工具类:
```Java
package utils;

import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

public class PasswordHashUtil {

    /**
     * 加密强度，不要超过 10 导致加密时间过久
     */
    private final static int STRENGTH = 10;

    /**
     * 生成密码 Hash
     */
    public static String encrypt(String password) {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(STRENGTH);
        return encoder.encode(password);
    }

    /**
     * 验证密码是否正确
     */
    public static boolean verifyPassword(String encodedPassword, String rawPassword) {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(STRENGTH);
        return encoder.matches(rawPassword, encodedPassword);
    }

}
```
<warning>
唯一需要注意的是，STRENGTH 不要设置的太高。其取值范围是 4 到 31，默认值为 10。一般 10 就可以了，如果设置为 16，在我的 Macbook Pro M1 上，加密需要花 4 秒的时间。
</warning>

## Test

单元测试如下:
```Java
package tests;

import utils.PasswordHashUtil;
import org.junit.Test;
import org.springframework.boot.test.context.SpringBootTest;

import static org.assertj.core.api.Assertions.*;

@SpringBootTest
public class TestPasswordHash {

    @Test
    public void match() {
        String password = "9kw@sgiWW8";
        String hash = PasswordHashUtil.encrypt(password);
        boolean pass = PasswordHashUtil.verifyPassword(hash, password);
        assertThat(pass).isEqualTo(true);
    }
}
```