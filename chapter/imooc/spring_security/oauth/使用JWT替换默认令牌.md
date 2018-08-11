# 使用JWT替换默认令牌

什么是jwt？

> JWT是json web token缩写。它将用户信息加密到token里，服务器不保存任何用户信息。服务器通过使用保存的密钥验证token的正确性，只要正确即通过验证。
>
>优点:在分布式系统中，很好地解决了单点登录问题，很容易解决了session共享的问题。
>
>缺点:是无法作废已颁布的令牌/不易应对数据过期。

特点：
1. 自包含 ： 包含自定义信息
2. 密签：使用指定密钥签名，防止串改，不是防止破解
3. 可扩展：也就是自定义业务信息

## 配置jwt

主要是增加了JwtTokenConfig类；
为了能方便切换，使用了 `@ConditionalOnProperty`注解；

```java
package cn.mrcode.imooc.springsecurity.securityapp;

@Configuration
public class TokenStoreConfig {
    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Bean
    @ConditionalOnProperty(prefix = "imooc.security.oauth2", name = "tokenStore", havingValue = "redis")
    public TokenStore tokenStore() {
        return new MyRedisTokenStore(redisConnectionFactory);
    }


    @Configuration
    // matchIfMissing ：当tokenStore没有值的时候是否生效
    // 当tokenStore = jwt的时候或则tokenStore没有配置的时候使用下面的配置
    @ConditionalOnProperty(prefix = "imooc.security.oauth2", name = "tokenStore", havingValue = "jwt", matchIfMissing = true)
    public static class JwtTokenConfig {
        @Autowired
        private SecurityProperties securityProperties;
        @Autowired
        private JwtAccessTokenConverter jwtAccessTokenConverter;

        @Bean
        public TokenStore jwtTokenStore() {
            return new JwtTokenStore(jwtAccessTokenConverter);
        }

        @Bean
        public JwtAccessTokenConverter jwtAccessTokenConverter() {
            JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
            converter.setSigningKey(securityProperties.getOauth2().getJwtSigningKey());  // 设置密钥
            return converter;
        }
    }
}

```
认证服务器还需要配置 JwtAccessTokenConverter
```java
@Autowired(required = false)
// 只有当使用jwt的时候才会有该对象
private JwtAccessTokenConverter jwtAccessTokenConverter;

@Override
public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
    endpoints.authenticationManager(this.authenticationManager);
    endpoints.tokenStore(tokenStore);
    if (jwtAccessTokenConverter != null) {
        endpoints.accessTokenConverter(jwtAccessTokenConverter);
    }
}
```

> 获取token：获取方式不变

```json
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp0aSI6IjkwYjQ4MTkxLTcwNGEtNDUxZS04NzkyLTk2NWMxYjNhMGMyMiIsImNsaWVudF9pZCI6Im15aWQiLCJzY29wZSI6WyJhbGwiXX0.bIe7RmyEaKdi8aX5C7JwaRq68m1WfpSXMvPZkjjoSus",
    "token_type": "bearer",
    "scope": "all",
    "jti": "90b48191-704a-451e-8792-965c1b3a0c22"
}
```
一个在线解码网址：http://jwt.calebb.net/ ; 把上面的access_token放进去

```json
{
 alg: "HS256",
 typ: "JWT"
}.
{
 user_name: "admin",
 jti: "90b48191-704a-451e-8792-965c1b3a0c22",
 client_id: "myid",
 scope: [
  "all"
 ]
}.
[signature]
```

访问；只是把之前的accessToken换成了jwt的accessToken信息
```
GET /user/me HTTP/1.1
Host: localhost:8080
Authorization: bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp0aSI6IjkwYjQ4MTkxLTcwNGEtNDUxZS04NzkyLTk2NWMxYjNhMGMyMiIsImNsaWVudF9pZCI6Im15aWQiLCJzY29wZSI6WyJhbGwiXX0.bIe7RmyEaKdi8aX5C7JwaRq68m1WfpSXMvPZkjjoSus

```


## TokenEnhancer jwt增强
因为Token的产生是框架流程出来的。我们如果需要在jwt生成之前对其修改；只能使用TokenEnhancer;

来源查看源码：`DefaultTokenServices#createAccessToken`中调用了TokenEnhancer

认证服务器配置，增加tokenEnhancer
```java
cn.mrcode.imooc.springsecurity.securityapp.MyAuthorizationServerConfig

    @Autowired(required = false)
   // 只有当使用jwt的时候才会有该对象
   private JwtAccessTokenConverter jwtAccessTokenConverter;
   /**
    * @see TokenStoreConfig
    */
   @Autowired(required = false)
   private TokenEnhancer jwtTokenEnhancer;


    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.authenticationManager(this.authenticationManager);
        endpoints.tokenStore(tokenStore);
        /**
         * 私有方法，但是在里面调用了accessTokenEnhancer.enhance所以这里使用链
         * @see DefaultTokenServices#createAccessToken(org.springframework.security.oauth2.provider.OAuth2Authentication, org.springframework.security.oauth2.common.OAuth2RefreshToken)
         */
        if (jwtAccessTokenConverter != null && jwtTokenEnhancer != null) {
            TokenEnhancerChain enhancerChain = new TokenEnhancerChain();
            List<TokenEnhancer> enhancers = new ArrayList<>();
            enhancers.add(jwtTokenEnhancer);
            enhancers.add(jwtAccessTokenConverter);
            enhancerChain.setTokenEnhancers(enhancers);
            // 一个处理链，先添加，再转换
            endpoints
                    .tokenEnhancer(enhancerChain)
                    .accessTokenConverter(jwtAccessTokenConverter);
        }
    }
```

TokenEnhancer 配置和自定义实现
```java
package cn.mrcode.imooc.springsecurity.securityapp;

/**
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/11 15:56
 */
@Configuration
public class TokenStoreConfig {
    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Bean
    @ConditionalOnProperty(prefix = "imooc.security.oauth2", name = "tokenStore", havingValue = "redis")
    public TokenStore tokenStore() {
        return new MyRedisTokenStore(redisConnectionFactory);
    }

    @Configuration
    // matchIfMissing ：当tokenStore没有值的时候是否生效
    // 当tokenStore = jwt的时候或则tokenStore没有配置的时候使用下面的配置
    @ConditionalOnProperty(prefix = "imooc.security.oauth2", name = "tokenStore", havingValue = "jwt", matchIfMissing = true)
    public static class JwtTokenConfig {
        @Autowired
        private SecurityProperties securityProperties;

        @Bean
        public TokenStore jwtTokenStore() {
            return new JwtTokenStore(jwtAccessTokenConverter());
        }

        @Bean
        public JwtAccessTokenConverter jwtAccessTokenConverter() {
            JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
            converter.setSigningKey(securityProperties.getOauth2().getJwtSigningKey());  // 设置密钥
            return converter;
        }

        @Bean
        // 不能使用该条件注解，因为JwtAccessTokenConverter也是一个TokenEnhancer
//        @ConditionalOnMissingBean(TokenEnhancer.class)
        // 而 ConditionalOnBean 是必须存在一个TokenEnhancer的时候，才被创建
        // 先不纠结这个问题了。就这样吧。也就是封装程度的问题
        @ConditionalOnBean(TokenEnhancer.class)
        public TokenEnhancer jwtTokenEnhancer() {
            return new ImoocJwtTokenEnhancer();
        }
    }
}

```

自定义增强器的实现
````java
/**
 *
 */
package cn.mrcode.imooc.springsecurity.securityapp.jwt;

import org.springframework.security.oauth2.common.DefaultOAuth2AccessToken;
import org.springframework.security.oauth2.common.OAuth2AccessToken;
import org.springframework.security.oauth2.provider.OAuth2Authentication;
import org.springframework.security.oauth2.provider.token.TokenEnhancer;

import java.util.HashMap;
import java.util.Map;

/**
 * @author zhailiang
 *
 */
public class ImoocJwtTokenEnhancer implements TokenEnhancer {
	@Override
	public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication) {
		Map<String, Object> info = new HashMap<>();
    // 需要增加的信息
    // 所以如果是需要动态的话，只能在该方法中去调用业务方法添加动态参数信息
		info.put("company", "imooc");

		// 设置附加信息
		((DefaultOAuth2AccessToken)accessToken).setAdditionalInformation(info);

		return accessToken;
	}
}

```

被解码后的值
```json
{
  alg: "HS256",
  typ: "JWT"
}.
{
  company: "imooc",    // 这里
  user_name: "admin",
  jti: "dcc5f820-e06f-4626-abc2-02e9cf7df8f8",
  client_id: "myid",
  scope: [
    "all"
  ]
}.
[signature]
```

这里有一个问题，spring的授权用户信息里面没有自定义的字段，所以要通过jwt的accessToken
获取到自定义信息的话，还需要自己来解析

可以跟下源码：解码是在这里
```java
org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter#decode

上面的代码能解析出自定义的信息，但是用户信息被 org.springframework.security.oauth2.provider.OAuth2Authentication 类最后返回的
。该类中没有自定义信息的字段。所以自定义信息丢失了
```
## 解析

security-demo/build.gradle 增加依赖；
为什么在demo里面？因为和上面分析的一致，公用的只管通用的，业务的自己处理
```
// ~ jwt==========================
compile 'io.jsonwebtoken:jjwt:0.9.1'
```
控制器中解析token信息
```
/**
 * 下面有几种获取方法，可以查看类里面的信息
 * @param userDetails
 * @param authentication
 * @param request
 * @return
 */
@GetMapping("/me")
public Object getCurrentUser(@AuthenticationPrincipal UserDetails userDetails, Authentication authentication, HttpServletRequest request) throws UnsupportedEncodingException {
//        Authentication authentication1 = SecurityContextHolder.getContext().getAuthentication();
    // Authorization : bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjb21wYW55IjoiaW1vb2MiLCJ1c2VyX25hbWUiOiJhZG1pbiIsImp0aSI6ImRjYzVmODIwLWUwNmYtNDYyNi1hYmMyLTAyZTljZjdkZjhmOCIsImNsaWVudF9pZCI6Im15aWQiLCJzY29wZSI6WyJhbGwiXX0.nYFBXcLBN3WNef0sooNxS0s6CaEleDGfjZh7xtTEqf4
    // 增加了jwt之后，获取传递过来的token
    // 当然这里只是其中一种的 token的传递方法，自己要根据具体情况分析
    String authorization = request.getHeader("Authorization");
    String token = StringUtils.substringAfter(authorization, "bearer ");
    logger.info("jwt token", token);
    String jwtSigningKey = securityProperties.getOauth2().getJwtSigningKey();
    // 生成的时候使用的是 org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter
    // 源码里面把signingkey变成utf8了
    // JwtAccessTokenConverter类，解析出来是一个map
    // 所以这个自带的JwtAccessTokenConverter对象也是可以直接用来解析的
    byte[] bytes = jwtSigningKey.getBytes("utf-8");
    Claims body = Jwts.parser().setSigningKey(bytes).parseClaimsJws(token).getBody();

    return body;
}
```
