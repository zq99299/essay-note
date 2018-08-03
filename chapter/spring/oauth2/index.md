# spring-oauth2

> 官网教程的一篇译文：https://www.cnblogs.com/xingxueliao/p/5911292.html
> 官网例子：https://github.com/spring-projects/spring-security-oauth/blob/master/samples/oauth2/sparklr/src/main/java/org/springframework/security/oauth/examples/sparklr/config/OAuth2ServerConfig.java

```
org.springframework.security.oauth2.provider.token.DefaultTokenServices#createAccessToken(org.springframework.security.oauth2.provider.OAuth2Authentication)  创建token的地方
发现有客户端模式的token有过期的话，就会创建一个刷新token，但是要怎么返回呢？

org.springframework.security.oauth2.provider.client.ClientCredentialsTokenGranter#grant 生成token的上层，也就是这里出现了一行注释
The spec says that client credentials should not be allowed to get a refresh token  ； 规范说不允许客户端获得刷新token；

这是为啥？

# 获取token
http://localhost:8080/oauth/token?grant_type=client_credentials&scope=select&client_id=client_1&client_secret=123456

# 刷新token
http://localhost:8080/oauth/token?grant_type=refresh_token&refresh_token=f6fa3c19-8201-4096-b4d8-a18369820e16&scope=select&client_id=client_1&client_secret=123456
```
