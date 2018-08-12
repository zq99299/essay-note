# 总结

> 本课程 练习代码：https://github.com/zq99299/spring-security.git

该课程讲解的是怎么写一个可重用的安全功能项目；

当然也学会了怎么使用和开发。

在 spring-security\doc 中总结成了文档。怎么使用这些安全模块；

也可以试着重新新建一个权限的项目，然后来引用这些安全模块，看能不能正常使用；

不要在之前的demo上面测试了，因为那个demo是从开发中而来，肯定没有什么问题了

这里是一些精华问题总结

## spring boot 启动扫描问题
之前被这个问题搞得很棘手；还百度了结果还是没有明白规则。只知道扫描

```java
// 注意看这里；demo启动类所在的包名
package com.imooc;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * @author zhailiang
 *
 */
@SpringBootApplication
@RestController
@EnableSwagger2
public class DemoApplication {

然后看一下其他模块的包名

package com.imooc.security.app.authentication;

package com.imooc;  对比下；

问题就是这里了。@SpringBootApplication 默认会扫描启动类所在的包下所有的类

如果不是可以手动指定扫描的包名路径

@SpringBootApplication(scanBasePackages =
        {
                "com.example.demo",
                "cn.mrcode.imooc.springsecurity.securitybrowser",
                "cn.mrcode.imooc.springsecurity.securityapp",
                "cn.mrcode.imooc.springsecurity.securitycore",
                "cn.mrcode.imooc.springsecurity.securityauthorize"
        })
```

## 之前那个AnyRequest的问题
解决思路：

1. 更改AuthorizeConfigProvider的接口返回值，
2. 然后在AuthorizeConfigManager的实现中去判定这个值

  当有没有配置AnyRequest的时候，自动添加一个；多个存在的时候报配置错误

```java
@Override
public void config(ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config) {
  boolean existAnyRequestConfig = false;
  String existAnyRequestConfigName = null;

  for (AuthorizeConfigProvider authorizeConfigProvider : authorizeConfigProviders) {
    boolean currentIsAnyRequestConfig = authorizeConfigProvider.config(config);
    if (existAnyRequestConfig && currentIsAnyRequestConfig) {
      throw new RuntimeException("重复的anyRequest配置:" + existAnyRequestConfigName + ","
          + authorizeConfigProvider.getClass().getSimpleName());
    } else if (currentIsAnyRequestConfig) {
      existAnyRequestConfig = true;
      existAnyRequestConfigName = authorizeConfigProvider.getClass().getSimpleName();
    }
  }

  if(!existAnyRequestConfig){
    config.anyRequest().authenticated();
  }
}
```

## 几句话

我开这门课程不是为了教你这么使用spring security，也不是怎么去写表单登录

想告诉你怎么去控制别人如何写代码，在一个公司中，如果你是一个开发，让别人来按照你的规则来写代码很有价值的一件事件；

spring 框架在08（某一年）被一家公司收购，卖了4亿美金，这是为什么？

因为它能控制全世界用这个框架的人怎么来写代码。这就是价值

在这门课程中演示的基本上都是，你要实现什么效果？需要去实现某一个接口，定制什么规则

## 最后

这是一个比较全的一个rbc应用，包括数据库表，页面，等。如果以后有用到可以去参考

https://github.com/zq99299/security/tree/master/imooc-security-authorize/src/main/java/com/imooc/security/rbac

听了老师一席话，感悟很深；陷入沉思； 完结
