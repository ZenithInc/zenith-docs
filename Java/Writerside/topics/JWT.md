# JWT

这篇文档描述了在 Java 中，如何使用第三方的 JWT 库。这里选择的是 `com.auth0.java-jwt` 这个库。其他的库也都类似，只是在 Java 中的 JWT 和其他语言中的库，在使用上略嫌麻烦，需要自己进一步封装。

## Install

我们可以通过 Maven 安装，配置如下:
```xml
<!-- https://mvnrepository.com/artifact/com.auth0/java-jwt -->
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.19.2</version>
</dependency>
```

## Generate

生成 JWT 的代码如下:
```Java
// 密钥和加密算法
String secretKey = "KNZcLhFvrkxu2ZdYDzzyJ4qYxKTiMrEt1zBbYkrSYzx";
Algorithm algorithm = Algorithm.HMAC256(secretKey);
// 计算过期时间
Calendar calendar = Calendar.getInstance();
Date now = calendar.getTime();
calendar.add(Calendar.SECOND, 7200);
// 生成 Token
String token = JWT.create()
	.withClaim("uid", 45657L)			// 业务参数，比如 userID
	.withExpiresAt(calendar.getTime())	// 过期时间
	.withIssuedAt(now)			// 签发时间
	.withIssuer("team.hacking.icu")		// 签发者
	.sign(algorithm);
System.out.println(token);
```

## Verify

验证 JWT 是否合法的代码如下:
```Java
String token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1aWQiOjQ1NjU3LCJpc3MiOiJ0ZWFtLmhhY2tpbmcuaWN1IiwiZXhwIjoxNjU0OTAzMjQzLCJpYXQiOjE2NTQ4OTYwNDN9.-2ZOgR1qhiqEF6_-DqHgWqLT3fUVGnevu9p5k1QS7HM";
Algorithm algorithm = Algorithm.HMAC256(secretKey);
DecodedJWT payload = JWT.require(algorithm).build().verify(token);
System.out.println(payload.getClaims().get("uid"));
```