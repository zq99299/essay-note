# qq登录下
前面把所有的代码组件都弄好了。现在可以开启调试了

在这之前你需要有一个qq互联的应用；也就是为了拿到appid和appSecret；自己去qq互联创建一个应用即可

这里讲下本地怎么调试应用

## 本地调试qq互联应用
申请好应用。并配置好
```
imooc:
  security:
    browser:
#      loginPage: /demo-signIn.html
#      loginType: REDIRECT
      loginType: JSON
    code:
      image:
        width: 100
        height: 50
        url: /order,/user/*
    social:
      qq:
        app-id: xxx
        app-secret: xxx
```

启动项目，qq登录，发现跳转到了qq登录的界面，登录完成后默认会跳转回 /signin地址

如果提示你回调地址验证失败；那么请验证,在qq互联的应用里面网站和回调地址是否设置好；

> 这是官网的教程：http://wiki.connect.qq.com/%E5%9B%9E%E8%B0%83%E5%9C%B0%E5%9D%80%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E5%8F%8A%E4%BF%AE%E6%94%B9%E6%96%B9%E6%B3%95
```
我的网站地址是：http://mrcode.cn/
回调地址就要写：http://mrcode.cn/
之前可以写根语名，但是现在不行了，必须写一个具体的回调地址；
所以回调地址需要修改成：http://mrcode.cn/outh/qq

/outh/qq 对应了social过滤器中的登录拦截地址，也就是我们前面在登录页面添加的提交地址；
```

那么这里就有一个小技巧了，怎么访问这个域名映射到我本地的项目呢？
有一个最简单的方法就是修改hosts文件

```
windows下hosts文件地址：C:\Windows\System32\drivers\etc

127.0.0.1 mrcode.cn
127.0.0.1 localhost

添加以上两个映射，保存文件
```
由于回调域名只支持80端口，本地项目端口也要修改成80端口；

现在就可以访问这个页面：http://mrcode.cn/imocc-signIn.html 会跳转到我们本地项目了

修改hosts原理：因为是在浏览器中进行跳转，改变的是浏览器的地址栏中的地址，
而mrcode.cn域名又被我们修改hosts文件映射到了本地，所以这样就可以调试qq登录了

## 如果你的回调url不是/outh/qq

那么需要覆盖配置; 并在之前配置new SpringSocialConfigurer 的地方修改成这里的覆盖之后的对象。即可
```java
package cn.mrcode.imooc.springsecurity.securitycore.social;

import org.springframework.social.security.SocialAuthenticationFilter;
import org.springframework.social.security.SpringSocialConfigurer;

/**
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/6 12:12
 */
public class MySpringSocialConfigurer extends SpringSocialConfigurer {
    @Override
    protected <T> T postProcess(T object) {
        // org.springframework.security.config.annotation.SecurityConfigurerAdapter.postProcess()
        // 在SocialAuthenticationFilter中配置死的过滤器拦截地址
        // 这样的方法可以更改拦截的前缀
        SocialAuthenticationFilter filter = (SocialAuthenticationFilter) super.postProcess(object);
        filter.setFilterProcessesUrl("/myouth");
        return (T) filter;
    }
}

```
