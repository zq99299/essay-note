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

## 基本流程源码解析
![](/assets/image/imooc/spring_secunity/snipaste_20180806_142855.png)

spring security的核心流程不变，social在这基础上增加了组件，和我们之前编写的流程类似；
上图蓝色部分是social除了security的东西外都是social实现好的。我们要做的只是 橘色部分；

当登录完成后，会调回 /outh/qq 并携带code；
```java
org.springframework.social.security.SocialAuthenticationFilter#attemptAuthService

org.springframework.social.security.provider.OAuth2AuthenticationService#getAuthToken

该方法处理了 是否携带code参数的 /outh/qq 的请求，如果没有则跳转到第三方授权页面。

这里在调用会拿code去换取accesstoken，调用api的时候解析出现了异常(根源码)
Could not extract response: no suitable HttpMessageConverter found for response type [interface java.util.Map] and content type [text/html]
抛出异常之后，会重定向到失败的地址，也就是/signin;

使用了的 org.springframework.web.client.RestTemplate 发送请求，
也就是之前我们在 cn.mrcode.imooc.springsecurity.securitycore.qq.connet.QQServiceProvider#QQServiceProvider 中提供的  OAuth2Template 类。里面就是使用的 RestTemplate

拿到响应结果之后会调用方法展开结果，
org.springframework.web.client.HttpMessageConverterExtractor#extractData；
在循环中没有找到对应的转换器，故而报错

MediaType contentType = getContentType(responseWrapper);

try {
  for (HttpMessageConverter<?> messageConverter : this.messageConverters) {
    if (messageConverter instanceof GenericHttpMessageConverter) {

```
稳定定位到了，那么就来处理掉，添加一个HttpMessageConverter进去应该就ok了；

那要怎么添加呢？最好就是查看下OAuth2Template是否有提供，没有的话就继承然后使用自己的
```java
package cn.mrcode.imooc.springsecurity.securitycore.qq.connet;

import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.converter.StringHttpMessageConverter;
import org.springframework.social.oauth2.AccessGrant;
import org.springframework.social.oauth2.OAuth2Template;
import org.springframework.util.MultiValueMap;
import org.springframework.web.client.RestTemplate;

import java.nio.charset.Charset;

/**
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/6 14:58
 */
public class QQAuth2Template extends OAuth2Template {
    private Logger logger = LoggerFactory.getLogger(getClass());

    public QQAuth2Template(String clientId, String clientSecret, String authorizeUrl, String accessTokenUrl) {
        super(clientId, clientSecret, authorizeUrl, accessTokenUrl);
        setUseParametersForClientAuthentication(true);
    }

    @Override
    protected RestTemplate createRestTemplate() {
        RestTemplate restTemplate = super.createRestTemplate();
        // 添加一个处理 [text/plan] 格式的转换器
        restTemplate.getMessageConverters().add(new StringHttpMessageConverter(Charset.forName("utf-8")));
        return restTemplate;
    }

    // http://wiki.connect.qq.com/%E4%BD%BF%E7%94%A8authorization_code%E8%8E%B7%E5%8F%96access_token
    // 文档中说明：响应的是 access_token=FE04************************CCE2&expires_in=7776000&refresh_token=88E4************************BE14
    // 不是一个json串
    @Override
    protected AccessGrant postForAccessGrant(String accessTokenUrl, MultiValueMap<String, String> parameters) {
        String responseStr = getRestTemplate().postForObject(accessTokenUrl, parameters, String.class);
        // 会发现返回的信息是 callback( {"error":100002,"error_description":"param client_secret is wrong or lost "} )
        // 通过debug可以发现，传递过来的参数少了2个，对比文档中的；
        // 调用本方法之前传递过来的参数，也就是 exchangeForAccess() 方法
        // 其中有一个 useParametersForClientAuthentication 属性需要为true才会携带另外另个参数
        logger.info("获取accessToken响应:{}", responseStr);
        String[] items = StringUtils.splitByWholeSeparatorPreserveAllTokens(responseStr, "&");
        String accessToken = StringUtils.substringAfterLast(items[0], "=");
        String expiresIn = StringUtils.substringAfterLast(items[1], "=");
        String refreshToken = StringUtils.substringAfterLast(items[2], "=");
        AccessGrant accessGrant = new AccessGrant(accessToken, null, refreshToken, new Long(expiresIn));
        return accessGrant;
    }
}

```

这其中出现很多问题，比如添加了转换器；一调试还是报错，只有一步一步的跟，
才会发现默认的OAuth2Template中是需要一个json串，并转成map，
然后又覆盖了这个获取解析的地方。

其他小地方也出现一些代码问题。那就是不要慌，保持本心，一步一步调试即可

回顾下流程

1. 访问/auth/qq,未携带code参数
2. 会重定向认证服务器，用户授权完成后，再调回原地址/auth/qq
3. social检测到携带了code参数，会去调用qqimpl交换accessToken
4. 条用api获取accessToken信息
5. 拿到令牌，包装成AccessGrant
6. 获取用户信息

但是这里获取完用户信息，就跳转到了 http://mrcode.cn/signup ； 注册页面

为什么会跳转到注册页面呢？下一节继续
