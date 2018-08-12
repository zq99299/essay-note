# 基于数据库Rbac数据模型控制权限
![](/assets/image/imooc/spring_secunity/snipaste_20180812_170316.png)

前面都是讲的怎么在权限规则基本不变的情况下，怎么写代码控制权限；

这一节要实现内管系统的场景；

这些所有的信息都必须存在数据库中。因为变动频繁，员工离职、部门调动，新增权限等；

## 通用RBAC数据模型
Role-Based-Access Control

通常由三直系表，两张关系表

对于资源表：存储数据的表现是 某一个url的别名是菜单或则按钮；所以url和多个菜单或则按钮绑定；

这样业务人员分配权限的时候才能看得懂

![](/assets/image/imooc/spring_secunity/snipaste_20180812_220236.png)

虽然是多对多，但是可以根据业务需要，进行一对多，比如一个用户只能拥有一个角色；

这个数据模型可以解决ui的显示问题和后台的程序权限控制

ui显示问题：提供一个查询该用户所有资源信息，然后由前端进行控制哪些资源显示或隐藏
url的程序控制：由security来处理

## 数据库中的数据如何交由security？

```java
// 任意请求都必须走自定义的表达式
// 该表达式的含义也很简单：rbacService 在容器中的beanName名称，后面的是方法名和参数名
config.anyRequest().access("@rbacService.hasPermission(request,authentication)")
```

只要有了入口，那么来实现这个功能；

新建一个项目，用来写这个表达式的功能；然后demo项目引用和配置表达式

security-authorize 依赖
```java
dependencies {
    compile 'javax.servlet:javax.servlet-api'
    // 核心的类都在该包中
    compile 'org.springframework.security:spring-security-core'
}
```
接口
```java
package cn.mrcode.imooc.springsecurity.securityauthorize;

import org.springframework.security.core.Authentication;

import javax.servlet.http.HttpServletRequest;

public interface RbacService {

    boolean hasPermission(HttpServletRequest request, Authentication authentication);
}

```
实现

```java
package cn.mrcode.imooc.springsecurity.securityauthorize;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;
import org.springframework.util.AntPathMatcher;

import javax.servlet.http.HttpServletRequest;
import java.util.HashSet;
import java.util.Set;

@Component("rbacService")
public class RbacServiceImpl implements RbacService {

    private AntPathMatcher antPathMatcher = new AntPathMatcher();

    @Override
    public boolean hasPermission(HttpServletRequest request, Authentication authentication) {
        Object principal = authentication.getPrincipal();

        boolean hasPermission = false;
        // 有可能 principal 是一个Anonymous
        // 所以只要是一个UserDetails那么就能标识是经过了我们自己的数据库查询的
        // 当前需要先配置UserDetailsServices
        if (principal instanceof UserDetails) {
            String username = ((UserDetails) principal).getUsername();
            //读取用户所拥有权限的所有URL
            Set<String> urls = new HashSet<>();
            for (String url : urls) {
                if (antPathMatcher.match(url, request.getRequestURI())) {
                    hasPermission = true;
                    break;
                }
            }

        }

        return hasPermission;
    }
}

```

demo项目中引用该包；并配置

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
                .permitAll();
        // 使用自定义的
        config.anyRequest().access("@rbacService.hasPermission(request,authentication)");
    }
}

```

注意：要把该包扫描到
```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@SpringBootApplication(scanBasePackages =
        {
                "com.example.demo",
                "cn.mrcode.imooc.springsecurity.securitybrowser",
                "cn.mrcode.imooc.springsecurity.securityapp",
                "cn.mrcode.imooc.springsecurity.securitycore",
                "cn.mrcode.imooc.springsecurity.securityauthorize"  // 注意添加扫描包
        })
//@SpringBootApplication
@RestController
@EnableSwagger2
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @GetMapping("/hello")
    public String hello() {
        return "hello spring security";
    }
}

```

ok。完成

## 其他问题

现在有两个问题需要解决：

1. 在通用的AuthorizeConfigProvider里面有一个 `config.anyRequest()`

  ```java
  return requestMatchers(ANY_REQUEST); 的源码，显示只能配置一个
  ```
2. `config.anyRequest()`的配置也只能在所有的配置完成之后，进行调用

### 第一个问题

先注释掉，通用AuthorizeConfigProvider里面的；

那么所有需要登录的配置要怎么办呢？视频中说后面总结的时候讲解；这里先留坑

### 第二个问题
使用order注解来解决，让我们的业务配置在最后；

第一次长见识，order还能用来控制 依赖查找的顺序

```java
@Component
public class DefaultAuthorizeConfigManager implements AuthorizeConfigManager {
    // 由于需要有序的，所以不能再使用set了
    // 依赖查找技巧
    @Autowired
    private List<AuthorizeConfigProvider> providers;

@Component
@Order(Integer.MIN_VALUE)
public class CommonAuthorizeConfigProvider implements AuthorizeConfigProvider {

@Component
@Order(Integer.MAX_VALUE)
public class DemoAuthorizeConfigProvider implements AuthorizeConfigProvider {

```
