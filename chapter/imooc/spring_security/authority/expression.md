# 权限表达式
看源码得知，最后都会转成一个表达式，然后进行投票评估；
那么有哪些表达式呢？

这些表达式的由来，由代码中的配置而来。
```java
.antMatchers().xxx  每个函数都包装了一个表达式生成。

跟着源码得到 返回的是一个  ExpressionUrlAuthorizationConfigurer.AuthorizedUrl 对象
```
![](/assets/image/imooc/spring_secunity/snipaste_20180812_203945.png)

联合使用是通过access方法，自己写表达式
```java
.antMatchers("xx").access("hasRole('ROLE_USER') and hasRole('ROLE_SUPER')")
```

那么能自定义表达式，并且使用自己的代码逻辑来判定吗？是可以的，下一节讲解；


## 分离配置

如下配置，一部分是安全模块的配置，一部分是使用安全模块的应用自己的业务配置；

那么怎么能把这种业务配置分离出去呢？
```java
.antMatchers(
         SecurityConstants.DEFAULT_UNAUTHENTICATION_URL,
         SecurityConstants.DEFAULT_LOGIN_PROCESSING_URL_MOBILE,
         "/user/regist", // 注册请求，后面会介绍怎么把这个只有使用方知道放行的配置剥离处理
         // org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController
         // BasicErrorController 类提供的默认错误信息处理服务
         "/error",
         "/connect/*",
         "/auth/*",
         "/signin"
 )
 .permitAll()
 // 该路径，只允许有 ADMIN 角色的人访问
 .antMatchers(HttpMethod.GET, "/user/*").hasRole("ADMIN")
```

![](/assets/image/imooc/spring_secunity/snipaste_20180812_205220.png)

思路：

* 提供AuthorizeConfigProvider接口
* 权限模块的通用配置实现该接口，然后进行配置
* 其他的应用或则模块配置可以自己实现
* 最后使用 AuthorizeConfigManager类来管理所有的AuthorizeConfigProvider实现
* 拿到所有的配置后，进行统一设置

接口定义
```java
package cn.mrcode.imooc.springsecurity.securitycore.authorize;

import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.ExpressionUrlAuthorizationConfigurer;

/**
 * 权限自定义配置管理
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/12 21:09
 */
public interface AuthorizeConfigManager {
    void config(ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config);
}
```

```java
package cn.mrcode.imooc.springsecurity.securitycore.authorize;

import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.ExpressionUrlAuthorizationConfigurer;

/**
 * 自定义权限控制接口
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/12 21:09
 */
public interface AuthorizeConfigProvider {
    /**
     * @param config
     * @see HttpSecurity#authorizeRequests()
     */
    void config(ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config);
}

```


核心实现： 通用配置的抽取，只是把app和browser中用到的配置都抽到公用的里面了
```java
package cn.mrcode.imooc.springsecurity.securitycore.authorize;

import cn.mrcode.imooc.springsecurity.securitycore.properties.SecurityConstants;
import cn.mrcode.imooc.springsecurity.securitycore.properties.SecurityProperties;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.ExpressionUrlAuthorizationConfigurer;
import org.springframework.stereotype.Component;

/**
 * app和browser通用静态权限配置
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/12 21:12
 */
@Component
public class CommonAuthorizeConfigProvider implements AuthorizeConfigProvider {
    @Autowired
    private SecurityProperties securityProperties;

    @Override
    public void config(ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config) {
        config.antMatchers(
                SecurityConstants.DEFAULT_UNAUTHENTICATION_URL,
                SecurityConstants.DEFAULT_LOGIN_PROCESSING_URL_MOBILE,
                SecurityConstants.DEFAULT_LOGIN_PROCESSING_URL_OPEN_ID,
                SecurityConstants.DEFAULT_VALIDATE_CODE_URL_PREFIX + "/*",
                securityProperties.getBrowser().getLoginPage(),
                securityProperties.getBrowser().getSignUpUrl(),
                securityProperties.getBrowser().getSession().getSessionInvalidUrl() + ".json",
                securityProperties.getBrowser().getSession().getSessionInvalidUrl() + ".html"
        ).permitAll();
        // 退出成功处理，没有默认值，所以需要判定下
        String signOutUrl = securityProperties.getBrowser().getSignOutUrl();
        if (signOutUrl != null) {
            config.antMatchers(signOutUrl).permitAll();
        }
    }
}
```

```java
package cn.mrcode.imooc.springsecurity.securitycore.authorize;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.ExpressionUrlAuthorizationConfigurer;
import org.springframework.stereotype.Component;

import java.util.Set;

/**
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/12 21:21
 */
@Component
public class DefaultAuthorizeConfigManager implements AuthorizeConfigManager {
    @Autowired
    private Set<AuthorizeConfigProvider> providers;

    @Override
    public void config(ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config) {
        for (AuthorizeConfigProvider provider : providers) {
            provider.config(config);
        }
        // 除了上面配置的，其他的都需要登录后才能访问
        config.anyRequest().authenticated();
    }
}

```

浏览器中的安全配置:
```java
// 有三个configure的方法，这里使用http参数的
@Override
protected void configure(HttpSecurity http) throws Exception {
    applyPasswordAuthenticationConfig(http);
    SessionProperties session = securityProperties.getBrowser().getSession();
    http
            .apply(validateCodeSecurityConfig)
            .and()
            .apply(smsCodeAuthenticationSecurityConfigs)
            .and()
            .apply(imoocSocialSecurityConfig)
            .and()
            .rememberMe()
            .tokenRepository(persistentTokenRepository)
            .tokenValiditySeconds(securityProperties.getBrowser().getRememberMeSeconds())
            .userDetailsService(userDetailsService)
            .and()
            .sessionManagement()
            .invalidSessionStrategy(invalidSessionStrategy)
            .maximumSessions(session.getMaximumSessions())
            .maxSessionsPreventsLogin(session.isMaxSessionsPreventsLogin())
            .expiredSessionStrategy(sessionInformationExpiredStrategy)
            .and()
            .and()
            .logout()
            .logoutSuccessHandler(logoutSuccessHandler)
            .deleteCookies("JSESSIONID")
            .and()
            .csrf()
            .disable();
    // 注入进来，然后把调用下配置对象即可
    // 可以看到上面的配置都没有了http.authorizeRequests()的配置
    // 全部由具体的去实现配置了
    // app项目中的安全配置改动其实和这里一样
    authorizeConfigManager.config(http.authorizeRequests());
}
```

demo项目的安全配置
```java
package com.example.demo.security;

import cn.mrcode.imooc.springsecurity.securitycore.authorize.AuthorizeConfigProvider;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.ExpressionUrlAuthorizationConfigurer;
import org.springframework.stereotype.Component;

/**
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/12 21:25
 */
@Component
public class DemoAuthorizeConfigProvider implements AuthorizeConfigProvider {
    @Override
    public void config(ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config) {
        config.antMatchers(
                "/user/regist", // 注册请求
                // org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController
                // BasicErrorController 类提供的默认错误信息处理服务
                "/error",
                "/connect/*",
                "/auth/*",
                "/signin",
                "/social/signUp",  // app注册跳转服务
                "/swagger-ui.html",
                "/swagger-ui.html/**",
                "/webjars/**",
                "/swagger-resources/**",
                "/v2/**"
        )
                .permitAll()
                // 这里配置了一个不存在的角色。
                // 可以访问下 看是否有效果
                .antMatchers("/user/*").hasRole("xxx")
        ;
    }
}
```
