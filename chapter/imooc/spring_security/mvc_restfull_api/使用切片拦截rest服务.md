# 使用切片拦截rest服务
本节内容

* 过滤器（Filter）
* 拦截器（interceptor）
* 切片（Aspect）

假设一个需求：打印出所有请求的耗时时间

## Filter

* 实现一个javax.servlet.Filter
* `@Component` 让这个实现类被spring容器接管

就可以让过滤器生效了
```java
package com.example.demo.web.filter;

import org.springframework.stereotype.Component;

import javax.servlet.*;
import java.io.IOException;
import java.time.Duration;
import java.time.Instant;

/**
 * ${desc}
 * @author zhuqiang
 * @version 1.0.1 2018/8/2 14:42
 * @date 2018/8/2 14:42
 * @since 1.0
 */
@Component  // 生效需要让spring容器接管
public class TimeFilter implements Filter {
    // 初始化
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("TimeFilter init");
    }

    // 执行
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        Instant start = Instant.now();
        chain.doFilter(request, response);
        System.out.println("耗时：" + Duration.between(start, Instant.now()).toMillis());
    }

    // 销毁
    @Override
    public void destroy() {
        System.out.println("TimeFilter destroy");
    }
}
```

## 编码注册过滤器

传统的过滤器可以使用 web.xml 等方式注册，spring boot 里面可以通过配置类添加

这种方式可以把一些第三方的过滤器添加进来

```java
package com.example.demo.web.config;

import com.example.demo.web.filter.TimeFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.servlet.Filter;

@Configuration  // 这里的注解别用错了
public class WebConfig {
    @Bean
    public FilterRegistrationBean timeFilter() {
        FilterRegistrationBean<Filter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new TimeFilter());
        // 可以自定义拦截路径
        registrationBean.addUrlPatterns("/*");
        return registrationBean;
    }
}
```

过滤器有一些限制，比如获取不到具体是哪一个方法处理的；

过滤器是j2ee的规范，在拦截器之前，还没有进入我们的具体控制器方法的时候被调用


## 拦截器（interceptor）

* 实现HandlerInterceptor拦截器
* 添加到spring kvc中

实现拦截器
```java
@Configuration
public class TimeInterceptor implements HandlerInterceptor {
    // 进入方法前
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        request.setAttribute("startTime", Instant.now());
        HandlerMethod method = (HandlerMethod) handler;
        System.out.println("preHandle " + method.getBean().getClass().getName());
        System.out.println("preHandle " + method.getMethodParameters());
        System.out.println("preHandle");
        return true;
    }

    // 进入方法后
    // 如果方法异常，则不会进入该节点
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        Instant startTime = (Instant) request.getAttribute("startTime");
        System.out.println("postHandle 耗时" + Duration.between(startTime, Instant.now()).toMillis());
    }

    // 请求后：无论如何都会走该节点
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

        System.out.println("afterCompletion");
        // 注意这里的异常，如果异常被全局异常处理器ControllerExceptionHandler消费掉了的话，这里的异常信息的null
        System.out.println("afterCompletion ex" + ex);
    }
}

```

添加到spring mvc中: 这个不同于过滤器的添加逻辑，需要手动进行配置
```java
@Configuration
// org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter 5.0+已过时
// 使用了jdk8 的接口默认方法
public class WebConfig implements WebMvcConfigurer {
    @Autowired
    private TimeInterceptor timeInterceptor;

    @Bean
    public FilterRegistrationBean timeFilter() {
        FilterRegistrationBean<Filter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new TimeFilter());
        // 可以自定义拦截路径
        registrationBean.addUrlPatterns("/*");
        return registrationBean;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 可以添加多个不同的拦截器
        registry.addInterceptor(timeInterceptor);
    }
}
```

## 切片（Aspect）
提供了更为强大的拦截功能，不只是能拦截mvc的handler；还能拦截符合截点的所有方法或则类

* 切入点（注解）
  - 在哪些方法上起作用
  - 在什么时候起作用
* 增强（方法）
  - 起作用时执行的业务逻辑

Aspect 注解属于aop包
`compile("org.springframework.boot:spring-boot-starter-aop")

> 官网能查看表切片怎么使用  和表达式是什么意思
> https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop

```java
@Component
@Aspect
public class TimeAspect {
    // 环绕通知 还有其他类型的注解
    // 这里的表达式在官网可以学习怎么使用
    // https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop
    @Around("execution(* com.example.demo.web.controller.UserController.*(..))")
    public Object doAccessCheck(ProceedingJoinPoint point) throws Throwable {
        Instant start = Instant.now();
        Object proceed = point.proceed();  // 类似于调用过滤器链一样
        // 这里对于异常来说和之前的都类似，异常的话下面不会继续走了
        System.out.println("耗时：" + Duration.between(start, Instant.now()).toMillis());
        Object[] args = point.getArgs();
        for (Object arg : args) {
            System.out.println(arg);
        }
        return proceed;
    }
}
```

## 总结
* 过滤器（Filter）
  - 能拿到最原始的http请求响应对象
  - 拿不到路径具体对于的handler
* 拦截器（interceptor）
  - 能拿到handler
  - spring家族成员
* 切片（Aspect）
  - 拿不到http请求响应对象
  - 可以拦截的更多：比如拦截mybatis生成的dao接口方法执行

处理顺序大概是下面这样；controllerAdvice是全局异常处理器
![](/assets/image/imooc/spring_secunity/snipaste_20180802_161748.png)
