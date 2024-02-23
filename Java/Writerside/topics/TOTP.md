# TOTP

这篇文档介绍了如何在项目中接入类似 Google Authenticator 这样的二次验证技术，提升项目中用户账户的安全性。

### 用户登陆二次验证技术

用户登录二次验证技术是一种增强账户安全性的方法。它要求用户在输入正确的用户名和密码后，再提供一个额外的验证因素，以确认用户的身份。以下是几种常见的二次验证技术：

1.  短信验证码：用户在输入用户名和密码后，会收到一条包含随机验证码的短信。用户需要将这个验证码输入到登录页面才能成功登录。

2.  身份验证应用程序：用户可以在手机上安装身份验证应用程序，如 Google Authenticator 或 Authy。这些应用程序生成一个基于时间的验证码，用户需要在登录页面输入正确的验证码才能完成登录。

3.  生物识别技术：一些设备和应用程序支持使用生物特征作为二次验证因素，如指纹识别、面部识别或虹膜扫描。用户在输入用户名和密码后，需要使用支持的生物识别技术进行身份验证。

4.  硬件安全密钥：硬件安全密钥是一种物理设备，可以通过USB或NFC与计算机或手机进行连接。用户在登录时需要将硬件安全密钥插入或靠近设备，以进行身份验证。

这些二次验证技术可以提供额外的安全层，防止未经授权的访问和账户被盗用。用户可以根据自己的需求和设备支持选择适合自己的二次验证方法。

### TOTP 基于时间的一次性密码算法

类似于 Google Authenticator 的软件采用了基于时间的一次性密码算法（Time-based One-Time Password，TOTP）。TOTP算法基于哈希函数和时间戳生成一次性密码。

具体来说，Google Authenticator使用HMAC-SHA1（Hash-based Message Authentication Code with Secure Hash Algorithm 1）算法对一个密钥和时间戳进行哈希运算，生成一个6位或8位的一次性密码。这个时间戳通常以30秒为间隔，以确保密码在一段时间后失效。

在用户登录时，Google Authenticator会生成一个基于当前时间的一次性密码，并在手机上显示给用户。用户需要在登录页面输入这个密码进行验证。服务器端也有相同的密钥，并根据当前时间和密钥计算出期望的一次性密码。只有在服务器端计算出的密码与用户输入的密码匹配时，登录才能成功。

TOTP算法的优点是简单、安全且易于实施。它不需要网络连接，可以在离线状态下生成密码，同时也能有效抵御钓鱼和重放攻击。

### TOTP 的工作流程 {id="work_flow"}

简要地描述Google Authenticator的交互过程：

1. 用户打开Google Authenticator应用程序，并与其账户进行绑定。
2. 在用户登录时，服务器将生成一个密钥（Secret Key）并将其发送给用户。
3. 用户将密钥输入到Google Authenticator应用程序中。
4. Google Authenticator应用程序使用密钥和当前时间戳生成一次性密码（One-Time Password，OTP）。
5. 生成的OTP显示在Google Authenticator应用程序中，并在一定时间后（30秒）过期。
6. 用户将OTP输入到登录页面进行验证。
7. 服务器使用相同的密钥和当前时间戳生成期望的OTP。
8. 服务器验证用户输入的OTP是否与期望的OTP匹配。
9. 如果匹配成功，用户登录成功；否则，登录失败。

需要注意的是: **类似于 Google Authenticator 的应用都是离线应用程序，我们的服务端生成密钥通过扫二维码或者输入的方式提交给 Google Authenticator。双方基于时间和固定的算法得出一个 6 位的验证码进行验证。**

换句话说：其实 Google Authenticator 只是一个离线应用，和我们的应用程序之间通过相同的密钥、相同的时间、相同的算法去计算相同的一次性密码。

### Java 实现 TOTP

为了加速开发速度，我们使用了下面这个 TOTP 的实现依赖, 可以通过 Maven 引入:
```xml
<!-- https://mvnrepository.com/artifact/com.eatthepath/java-otp -->
<dependency>
    <groupId>com.eatthepath</groupId>
    <artifactId>java-otp</artifactId>
    <version>0.4.0</version>
</dependency>
```
这里的关键在于，每一个用户账号在绑定验证器(Google Authenticator)之前，都需要事先生成一个 SecretKey，生成后需要保存到数据库，验证的时候还需要用：
```java
KeyGenerator generator = KeyGenerator.getInstance("HmacSHA1");
int macLengthInBytes = Mac.getInstance("HmacSHA1").getMacLength();
generator.init(macLengthInBytes * 8);
SecretKey secretKey = generator.generateKey();
String sk = new Base32().encodeAsString(secretKey.getEncoded());
System.out.println("sk = " + sk);       // 例如: QXWVOYIP6SJTAGZX2JKV2FECHZYN2TIP
```
> 需要强调的是: **在 TOTP 中，是通过 Base32 编码而不是 Base64 ，这非常容易踩坑，而且不容易排查！**

然后，服务端通过如下代码生成当前的验证码，和用户输入的验证码进行对比即可:
```java
String secretKey = "QXWVOYIP6SJTAGZX2JKV2FECHZYN2TIP";
TimeBasedOneTimePasswordGenerator generator = new TimeBasedOneTimePasswordGenerator();
byte[] decodedKey = new Base32().decode(secretKey);
String currentCode = generator.generateOneTimePasswordString(
        new SecretKeySpec(decodedKey, 0, decodedKey.length, "HmacSHA1"),
        Instant.now()
);
System.out.println("currentCode = " + currentCode);     // 087543
```
### 绑定二维码生成

绑定二维码的生成，可以采用国外的一个专业的二维码生成网站接口，这个接口是免费的，而且在不恶意使用的情况下，是不会限制调用频率的。

后端拼接返回前端下面这个 URL，或者返回 SecretKey 前端自定拼接即可:
```java
// 拼接字符串
otpauth://totp/<系统或用户信息>?secret=<密钥>
// 将上面的字符串拼接到下面的 data 参数中
http://api.qrserver.com/v1/create-qr-code/?data=<此处替换>!&size=250x250
```

其中 `QXWVOYIP6SJTAGZX2JKV2FECHZYN2TIP` 是我们之前生成的 SecretKey。可以通过 URL 中的 size 参数指定生成的二维码图片大小。

### 其他安全措施

在实现的过程中，还需要进一步增强安全性，特别是在绑定流程上。可以参考以下措施, 这些在 CAS 项目中应用:

1. Secret Key 只能返回客户端一次，即在绑定的时候；
2. 因为第 1 点，所以绑定的二维码也只能显示一次，在此刷新的时候也不能包含 Secret Key；
3. 所以处于安全考虑，只能绑定单一验证器。要绑定新的验证器之前，需要将之前的 Secret Key 作废。
4. 如果从需求上用户需要绑定多个设备，需要每次都采用新的 Secret Key。
5. 对验证码错误的次数做出限制，比如错误 5 次，冻结账户，否则有暴力破解风险。

### 总结

这篇文档介绍了用户登陆二次验证技术的多种实现，详细讲解了 TOTP 这种基于时间的一次性密码算法。接着介绍了 Google Authenticator 验证器和我们项目验证的交互流程。最后，文档给出了 TOTP 在 Java 中的实现应用，二维码的简单生成。

安全从来不是绝对的，所以，及时使用了二次验证，也要反复思考每一个环节是否有漏洞。增加更多的安全措施。
