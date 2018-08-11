# 基于JWT实现SSO单点登录1

single sign on(SSO) 的效果是什么？

![](/assets/image/imooc/spring_secunity/snipaste_20180811_204615.png)

如上图：

1. 用户在应用a触发了登录，那么a会拿到一个jwt信息
2. 用户在应用b不用登录，会发现已经登录过了?然后返回一个jwt给应用b。完成应用b的登录

这里没有搞明白是怎么控制的、后面再来完善

## 创建项目结构

不在之前的项目上继续了，之前的项目用于讲解浏览器和app不同的支持；

这次的是既支持浏览器跳转也支持json返回这种。

编写的代码比较简单。从零开始搭建sso需要做些什么。更容易理解；

如果自己有任何需求，可以自定义的去实现合并某些功能

项目结构：分别对应上图的4个角色；
```
sso-client1
sso-client2
sso-demo
sso-server
```

4个都是gradle的模块
```
rootProject.name = 'spring-security'
include 'security-app'
include 'security-browser'
include 'security-core'
include 'security-demo'
include 'sso-client1'
include 'sso-client2'
include 'sso-demo'
include 'sso-server'
```

## 认证服务器的搭建
sso-server/build.gradle
```
dependencies {
	compile('org.springframework.boot:spring-boot-starter-security')
	compile('org.springframework.boot:spring-boot-starter-web')
	compile 'org.springframework.security.oauth:spring-security-oauth2'
	compile 'org.springframework.security:spring-security-jwt'
	testCompile('org.springframework.boot:spring-boot-starter-test')
	testCompile('org.springframework.security:spring-security-test')
}

```

对比下面的配置：

这里的配置与之前的core中的配置最大的区别是，没有引入 org.springframework.security.oauth.boot:spring-security-oauth2-autoconfigure:2.0.0.RELEASE
可以是因为这个原因，才不需要设置authenticationManager吧？

```java

package cn.mrcode.imooc.springsecurity.sso.ssoserver;

@Configuration
@EnableAuthorizationServer
public class SsoAuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("myid1")
                .secret("myid1")
                .authorizedGrantTypes("authorization_code", "refresh_token")
                .scopes("all")
                .redirectUris(
                        "http://example.com",
                        "http://ora.com")
                .and()
                .withClient("myid2")
                .secret("myid2")
                .authorizedGrantTypes("authorization_code", "refresh_token")
                .scopes("all");
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.tokenStore(jwtTokenStore()).accessTokenConverter(jwtAccessTokenConverter());
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        // 客户端来向认证服务器获取签名的时候需要登录认证身份才能获取
        // 因为客户端需要用密钥解密jwt字符串
        security.tokenKeyAccess("isAuthenticated()");
    }

    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey("imooc");
        return converter;
    }
}

```
注意改成默认的basic；
```java
package cn.mrcode.imooc.springsecurity.sso.ssoserver;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
public class MyWebSecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.httpBasic();
    }
}

```

sso-server application.yml
```yml
server:
  port: 9999
  servlet:
    context-path: /server

spring:
  security:
    user:
      password: 123456
```

启动项目，一切正常；

浏览器访问以下地址获取code;发现连授权页面都和以前的不一样了。我有点费解

```
http://localhost:9999/server/oauth/authorize?response_type=code&client_id=myid1&redirect_uri=http://www.example.com&scope=all
```
