# 个人实战技巧

学习始终是学习，使用在项目中的时候，总会发现一些需求不太满足，那么这个时候就只能靠我们自己阅读源码了，
本文记录一些视频中没有讲到的东西，或则是一些比较好的思路

## 携带 access_token 访问发现过期了，要要捕获这个动作

场景：登录认证获取到了有效的 access_token ，但是在访问中，发现失效了，目前 oath2 默认的处理方法是
抛出异常 org.springframework.security.oauth2.common.exceptions.InvalidTokenException  `Access token expired: 13f43f2d-27b2-49b3-82f5-cf31838ef79b`

比如访问：`http://localhost:9504/api/life-cyle/task?access_token=13f43f2d-27b2-49b3-82f5-cf31838ef79b` 页面打印的就是这一串信息。

postman 中打印的是这样的信息

```javascript
{
    "error": "invalid_token",
    "error_description": "Access token expired: 13f43f2d-27b2-49b3-82f5-cf31838ef79b"
}
```

需求来了：假如我们的策略是这样的：

暴露在外的只有 网关，我们的授权认证在网关上。

1. 当用户认证成功之后，我们把 key=token,value=简单的用户信息，放入了 redis 中
2. 其他项目获取到 token 后，去 resdis 中获取信息，

**问题来了：** 当过期之后这个 token 信息怎么移除？

**问题原因：**

* 每次获取有效的 token 都是不一样的
* 无法保证设置多长时间的过期时间，也有可能会有主动移除 tokenService 中的 token 信息的情况。

那么我们的救命稻草之一就是，在这个检查过期的地方进行触发删除掉 redis 中的信息；

跟踪源码发现，在异常的时候推送了一个异常事件。有 event 那么我们能不能接收呢？

```java
org.springframework.security.oauth2.provider.authentication.OAuth2AuthenticationProcessingFilter#doFilter

catch (OAuth2Exception failed) {
  SecurityContextHolder.clearContext();

  if (debug) {
    logger.debug("Authentication request failed: " + failed);
  }
  eventPublisher.publishAuthenticationFailure(new BadCredentialsException(failed.getMessage(), failed),
      new PreAuthenticatedAuthenticationToken("access-token", "N/A"));

  authenticationEntryPoint.commence(request, response,
      new InsufficientAuthenticationException(failed.getMessage(), failed));

  return;
}

chain.doFilter(request, response);

------------------------------

org.springframework.security.authentication.DefaultAuthenticationEventPublisher#publishAuthenticationFailure

// 又委托了 applicationEventPublisher
if (event != null) {
  if (applicationEventPublisher != null) {
    applicationEventPublisher.publishEvent(event);
  }
}
```

根据源码发现委托了 applicationEventPublisher，那么这里我就联想到了，spring 的 `@EventListener` 注解，可以捕获一些事件

> https://docs.spring.io/spring/docs/5.1.3.BUILD-SNAPSHOT/spring-framework-reference/core.html#context-functionality-events-annotation

### 问题解决

```java
/**
 * 事件处理
 * @author zhuqiang
 * @version 1.0.1 2018/10/31 16:50
 * @date 2018/10/31 16:50
 * @since 1.0
 */
@Component
@Slf4j
public class ApplicationEventHandler {
    /**
     * token 认证无效的时候处理：
     * 包括：
     * - 过期 Access token expired
     * - 无效 Invalid access token
     */
    @EventListener(AuthenticationFailureBadCredentialsEvent.class)
    public void authenticationFailureBadCredentialsEvent(AuthenticationFailureBadCredentialsEvent event) {
        log.info(event.toString());
    }
}
```

在源码中是根据 异常类的 Name 创建的 反射创建的 event。 这里的异常都是 BadCredentialsException 。所以只能捕获该事件

他的父类 AbstractAuthenticationFailureEvent 有很多细分的事件，包括过期的，但是目前没有发现怎么捕获；

捕获之后，就可以 移除 redis 中的信息了；
