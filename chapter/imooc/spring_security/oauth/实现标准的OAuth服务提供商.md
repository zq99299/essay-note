# 实现标准的OAuth服务提供商

写在app中，所以demo项目的依赖需要修改下

```
dependencies {
//    compile project(':security-browser') // 开发app，先暂时注释掉
    compile project(':security-app')

本次依赖更改出错的地方有：

cn.mrcode.imooc.springsecurity.securitycore.validate.code.ValidateCodeFilter
中需要两个处理器，在app中先复制一份出来

com.example.demo.security.MyUserDetailsService#passwordEncoder
passwordEncoder 之前写在browser的，抽取到core里面
```

## 依赖
```java
// security自动配置
// 以及包含了 spring-cloud-start,spring-cloud-security 、spring-boot-starter-actuator
// security 5+ 去掉了可以在配置文件中关闭security的配置，所以这里在视频中配置关闭的时候
// 我们在这里注释掉依赖就可以了
//    compile('org.springframework.cloud:spring-cloud-starter-security')
// 多包涵了一个spring-security-oauth2-autoconfigure
compile('org.springframework.cloud:spring-cloud-starter-oauth2')
```


## 认证服务器
```java
package cn.mrcode.imooc.springsecurity.securitycore;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;

@Configuration
@EnableAuthorizationServer  // 添加一个配置类即可
public class MyAuthorizationServerConfig {
}

```

启动项目会自动增加以下几个控制器
```java
"{[/oauth/authorize]}"
"{[/oauth/authorize],methods=[POST],
"{[/oauth/token],methods=[GET]}"
"{[/oauth/token],methods=[POST]}"
"{[/oauth/check_token]}"
"{[/oauth/confirm_access]}"
"{[/oauth/error]}"
```
且在控制台会打印一个默认的clientid(每次都动态生成)
```
security.oauth2.client.client-id = 666b1e3c-bbec-4a6c-86ca-3387dd113519
security.oauth2.client.client-secret = eea6a558-ce58-4d82-b553-b70406005c8b

可以修改成固定的，方便后面的调试

security:
  oauth2:
    client:
      client-id: myid
      client-secret: myid
```


>官网文档  https://tools.ietf.org/html/rfc6749#section-4

由于spring oath2实现的是标准的oat2协议，所以参数什么的一般可以参考官网文档，如上链接。
获得授权部分

访问以下地址缺报错了。
```
http://localhost:8080/oauth/authorize?response_type=code&client_id=myid&redirect_uri=http://www.example.com&scope=all

There was an unexpected error (type=Internal Server Error, status=500).
User must be authenticated with Spring Security before authorization can be completed.

不知道为什么一直走
org.springframework.security.web.authentication.AnonymousAuthenticationFilter#doFilter

在这个报错的地方很容易找到，里面说必须要经过security的安全认证。

现在终于串联起来了。之前看到过一篇文章在WebSecurityConfigurerAdapter配置类中打开了运行表单认证
然后就可以访问了。原来是这样。视频中的版本是直接默认basic认证的，所以不需要配置什么；
@Override
       public void configure(AuthorizationServerSecurityConfigurer oauthServer) throws Exception {
           //允许表单认证
           oauthServer.allowFormAuthenticationForClients();
       }
```

这里收必须要经过security认证才可以，视频中是直接跳出来一个 basic登录框。

** 重要的事情说三遍：security5+ 认证默认为表单了也就是http.formLogin() **

** 重要的事情说三遍：security5+ 认证默认为表单了也就是http.formLogin() **

** 重要的事情说三遍：security5+ 认证默认为表单了也就是http.formLogin() **

所以这里还需要把security的默认表单登录改成basic登录

```java
@Configuration
public class MyWebSecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.httpBasic();
    }
}
```

再次登录再次报错：
```
error="invalid_request", error_description="At least one redirect_uri must be registered with the client."
``

报错点
```java
org.springframework.security.oauth2.provider.endpoint.DefaultRedirectResolver#resolveRedirect

/**
 * The pre-defined redirect URI for this client to use during the "authorization_code" access grant. See OAuth spec,
 * section 4.1.1.
 *
 * @return The pre-defined redirect URI for this client.
 */
Set<String> getRegisteredRedirectUri();
```
打开看了下规范，也没有太看明白。应该和qq登录那边一样的，需要设置一个授权回调域

而在代码中报错的地方调试最后发现
```java
org.springframework.boot.autoconfigure.security.oauth2.authserver.OAuth2AuthorizationServerConfiguration.BaseClientDetailsConfiguration#oauth2ClientDetails

@Bean
		@ConfigurationProperties(prefix = "security.oauth2.client")
		public BaseClientDetails oauth2ClientDetails() {
			BaseClientDetails details = new BaseClientDetails();
			if (this.client.getClientId() == null) {
				this.client.setClientId(UUID.randomUUID().toString());
			}
			details.setClientId(this.client.getClientId());
			details.setClientSecret(this.client.getClientSecret());
			details.setAuthorizedGrantTypes(Arrays.asList("authorization_code",
					"password", "client_credentials", "implicit", "refresh_token"));
			details.setAuthorities(
					AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_USER"));
			details.setRegisteredRedirectUri(Collections.<String>emptySet());
			return details;
		}
```

也就是说需要给client配置回调域

```yml
security:
  oauth2:
    client:
      client-id: myid
      client-secret: myid
      registered-redirect-uri:
        - "http://example.com"
        - "http://ora.com"
```

再次访问，出现了久违的需要同意授权的页面，授权后，跳转到了
```
http://example.com/?code=nayO4i
```

## 密码授权模式

访问
`http://localhost:8080/oauth/token?grant_type=password&username=admin&password=123456&scope=all`

弹出了basic的登录框，后台报错
```java
org.springframework.security.access.AccessDeniedException: Access is denied  

访问被拒绝

原因就是:在demo里面配置了一个userdetails

private SocialUser getUserDetails(String username) {
    String password = passwordEncoder.encode("123456");
    logger.info("数据库密码{}", password);
    SocialUser admin = new SocialUser(username,
//                              "{noop}123456",
                                      password,
                                      true, true, true, true,
                                      AuthorityUtils.commaSeparatedStringToAuthorityList("admin,ROLE_USER"));
    return admin;
}

需要添加一个  ROLE_USER 的角色；
```



## 资源服务器
```java
package cn.mrcode.imooc.springsecurity.securitycore;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;

@Configuration
@EnableResourceServer
public class MyResourcesServerConfig {
}

```

访问任何信息都会报错:`Full authentication is required to access this resourceunauthorized`
是因为没有携带token。



> 到现在都没有抛出来看到效果。也不知道是哪里的问题诶,可能是要真的配置 security才可以
>
