# 单机session管理

到目前为止三个功能：

* 用户名 + 密码登录
* 手机号 + 短信登录
* 社交网站登录

前两种使用表单提交方式完成，后一种使用oath授权完成；

虽然表现方式和处理流程不同，但是有一个共同点，认证后的用户信息是存放在session中的；


* session超时
  - 如何管理超时时间
  - 超时后如何处理
* session并发 ： a 机器登录，又在b机器登录的场景下，只运行一台机器登录
  - 如何保持后来者生效，之前的失效

* 集群session管理
  均衡负载如果没有做session粘连的话，会出现登录在a机器，请求数据在b机器

## session超时
> 注意：server.session.timeout 已经过时了
```yml
server:
  port: 80
  servlet:
    session:
      timeout: 10s
```
重启测试，发现并没有超时；在以下源码中限制了最小为1分钟；

```java
org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory#configureSession
```
## 实现session超时提醒
session超时之后，再次访问进行一个提醒要怎么做呢？

```java
cn.mrcode.imooc.springsecurity.securitybrowser.BrowserSecurityConfig#configure

配置下session失效跳转的url地址，这个地址需要我们实现。你可以做任何的业务逻辑;同时记得放行该地址，否则又被拦截授权了
.and()
.sessionManagement()
.invalidSessionUrl("/session/invalid")
```

```java
cn.mrcode.imooc.springsecurity.securitybrowser.BrowserSecurityController#sessionInvalid
这里就打印下消息

@GetMapping("/session/invalid")
@ResponseStatus(HttpStatus.UNAUTHORIZED)
public SimpleResponse sessionInvalid() {
    String message = "session失效";
    return new SimpleResponse(message);
}
```

## 实现session并发登录控制

```java
// 配置地址或则策略类
.maximumSessions(1) //限制同一个用户只能有一个session登录
//.expiredUrl("/session/expired") // 也可以跳转到一个服务
.expiredSessionStrategy(new MySessionInformationExpiredStrategy())  // 失效后的策略。定制型更高，失效前的请求还能拿到
```
编写策略类
```java
/**
 * session并发登录失效策略
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/6 21:28
 */
public class MySessionInformationExpiredStrategy implements SessionInformationExpiredStrategy {
    @Override
    public void onExpiredSessionDetected(SessionInformationExpiredEvent event) throws IOException, ServletException {
        // 该对象能获取到访问失效前的url地址
        event.getResponse().setContentType("application/json;charset=UTF-8");
        event.getResponse().getWriter().write("session并发登录");
    }
}
```

测试：在不同浏览器登录，然后在最开始登录的浏览器中访问一个服务查看下；

## 实现session并发登录策略2

```java
.maximumSessions(1) //限制同一个用户只能有一个session登录
.maxSessionsPreventsLogin(true)  // 当session达到最大后，阻止后登录的行为
```
测试会提示："Maximum sessions of 1 for this principal exceeded"

## 代码重构，消除重复代码，可提供可配置功能

这里尝试自己看一遍视频，然后全程去思考如何提出来。不行的时候再看源码

重构的时候需要注意一个坑：
```java
/** 当session达到最大值后，是阻止用户登录还是剔除掉已登录用户
 * fasle ： 会走{@link cn.mrcode.imooc.springsecurity.securitybrowser.session.MySessionInformationExpiredStrategy}
 * true：会阻止登录，这个阻止登录的个性化消息没有设置，看源码的时候好像可以覆盖那个过滤器；设置为true会看到报错信息，然后就可以查看覆盖说明了
 * */
private boolean maxSessionsPreventsLogin;
```
