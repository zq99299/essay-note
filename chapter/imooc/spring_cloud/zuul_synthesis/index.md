# Zuul综合使用 上

* pre 过滤器
* post过滤器
* zuul 限流

![](/assets/image/imooc/spring_cloud/snipaste_20180819_172123.png)

这个是这次实战的整体架构图

可以看到，所有的请求都会经过zuul

## pre 过滤器
实现这样一个场景：包含了 token 参数的才允许访问

```java
package cn.mrcode.imooc.spring.cloud.apigateway;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.cloud.netflix.zuul.filters.pre.PreDecorationFilter;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

import javax.servlet.http.HttpServletRequest;

/**
 * 权限拦截认证：简单判定下是否带有 token参数
 * 可以参考 {@link PreDecorationFilter}
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/19 17:25
 */
@Component
public class TokenFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.PRE_DECORATION_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        // 返回true，则意味着调用 run方法
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        // 可以从cookie，header里获取
        String token = request.getParameter("token");
        if (StringUtils.isEmpty(token)) {
            // 不通过
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(HttpStatus.UNAUTHORIZED.value());
            try {
                ctx.getResponse().getWriter().write("token is empty");
            } catch (Exception e) {
            }
        }
        return null;
    }
}

```

## post过滤器

往返回的结果里面添加东西;

这里测试到一个现象: 无论前置过滤器是否通过，后置过滤器都会被执行

```java
package cn.mrcode.imooc.spring.cloud.apigateway;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * 往返回头中添加参数
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/19 17:41
 */
@Component
public class AddResponseHeaderFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return FilterConstants.POST_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.SEND_RESPONSE_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        ctx.getResponse().addHeader("x-mrcode", new Date().getTime() + "");
        return null;
    }
}

```

## zuul 限流

做限流保护，如做发短信的，需要控制api的请求速度

时机：请求被转发之前调用

![](/assets/image/imooc/spring_cloud/snipaste_20180819_174803.png)

令牌桶限流：

1. 固定速率把令牌放入桶中
2. 当桶中满的时候则会丢弃掉
3. 请求到达时，会向桶中获取令牌，如果获取到了才能继续访问

这个原理看起来地区是很方便的。一个地方固定速率产生令牌，也就相当于控制整体的请求速度了？

```java
package cn.mrcode.imooc.spring.cloud.apigateway;

import com.google.common.util.concurrent.RateLimiter;
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;

/**
 * 令牌桶限流过滤器
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/19 17:53
 */
@Component
public class RateFilter extends ZuulFilter {
    // com.google.common.util.concurrent.RateLimiter 谷歌的一个令牌桶算法
    // 每秒放入两个令牌
    private RateLimiter rateLimiter = RateLimiter.create(2);

    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.SERVLET_DETECTION_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        // 尝试获取令牌
        if (rateLimiter.tryAcquire()) {
            throw new RuntimeException("限流异常");
        }
        return null;
    }
}

```

> 有一个开源的限流实现，支持多种数据源，redis等
>
> https://github.com/marcosbarbero/spring-cloud-zuul-ratelimit
