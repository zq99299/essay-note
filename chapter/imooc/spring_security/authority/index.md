# Spring Security授权简介

授权又是什么概念呢？

现在来回顾下安全的概念：
1. 你是谁？
2. 你能干什么？

前面讲解的全是认证，也就是解决你是谁的问题；

这章讲解你能干什么的问题。很多人叫权限控制，鉴权，授权等；最终的核心目的都是一样的，
控制这个用户能在系统中干什么？

## security对授权的定义
![](/assets/image/imooc/spring_secunity/snipaste_20180812_152323.png)

上图意思就是说，页面能看到的只是体验和ui交互问题；而对应后台某一个url的是否能被访问被认为是权限

## 权限场景分析
![](/assets/image/imooc/spring_secunity/snipaste_20180812_170316.png)

两个系统的权限特点是不一样的。不应该部署在一个应用中

![](/assets/image/imooc/spring_secunity/snipaste_20180812_170637.png)

当权限比较简单的时候，也就是对应业务系统来说这种需求；

security就支持了，可以把规则写在代码中进行控制

## security的权限控制
之前其实已经写过
```java
// 对请求授权配置：注意方法名的含义
 .authorizeRequests()
 .antMatchers(
         SecurityConstants.DEFAULT_UNAUTHENTICATION_URL,
         SecurityConstants.DEFAULT_LOGIN_PROCESSING_URL_MOBILE,
         securityProperties.getBrowser().getLoginPage(),
         SecurityConstants.DEFAULT_VALIDATE_CODE_URL_PREFIX + "/*", // 图形验证码接口
         securityProperties.getBrowser().getSignUpUrl(),  // 注册页面
         securityProperties.getBrowser().getSession().getSessionInvalidUrl() + ".json",
         securityProperties.getBrowser().getSession().getSessionInvalidUrl() + ".html",
         "/user/regist", // 注册请求，后面会介绍怎么把这个只有使用方知道放行的配置剥离处理
         "/error",
         "/connect/*",
         "/auth/*",
         "/signin"
 )
  // 放行以上路径
 .permitAll()
 // 该路径，只允许有 ADMIN 角色的人访问
 .antMatchers("/user").hasRole("ADMIN")
 .anyRequest()
 // 对任意请求都必须是已认证才能访问
 .authenticated()
```
这个时候再访问系统，登录后发现 /user 不能访问了;访问被拒绝了，那么就是因为当前登录的用户没有 ADMIN这个角色

```
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Sun Aug 12 17:23:47 CST 2018
There was an unexpected error (type=Forbidden, status=403).
Forbidden
```

## 如何给用户分配角色

就是在我们之前实现的UserDetailsService来进行构建的用户信息  
```java
com.example.demo.security.MyUserDetailsService

private SocialUser getUserDetails(String username) {
    String password = passwordEncoder.encode("123456");
    logger.info("数据库密码{}", password);
    SocialUser admin = new SocialUser(username,
//                              "{noop}123456",
                                      password,
                                      true, true, true, true,
                                      AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_ADMIN"));
    return admin;
}
```

这里的角色需要添加 ROLE 前缀；因为之前使用的是 hasRole ； 这个在下一章节讲解是为什么；


## 怎么匹配restfull的url

还是之前那个方法配置，可以传递请求和通配符
```java
.antMatchers(HttpMethod.GET, "/user/*").hasRole("ADMIN")
```

在权限规则简单的情况下，就可以使用这里的知识进行构建权限系统。

下一章：讲解一个url请求，security对它做了些什么
