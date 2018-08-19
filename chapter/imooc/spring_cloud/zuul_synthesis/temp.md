# 业务使用

* 使用过滤器，传统方式进行权限判定
* 跨域介绍

## zuul权限校验

* /order/create （下单）  只能买家访问
* /order/finish （结单） 只能卖家访问
* /product/list   都可以访问

还是使用过滤器，

1. 在登录后给用户设置cookie，买家为openid，卖家为token（并存储在redis中）
2. 在访问的时候获取该cookie，先从redis中命中，否则认为是买家，最后从数据库获取信息
3. 判定权限，是否放行

这个业务逻辑基本上没有贴代码的必要；

有一个边界的问题：

网关不能做复杂的业务。否则网关边界就有问题了

## 跨域

* 跨域问题
* 在被调用的类或方法上增加`@CorsOrigin注解` : 对单个接口跨域设置
* 在Zuul里增加CorsFilter ： 对全局配置

// spring 的CorsFilter过滤器。只不过配置在了zuul项目中
```java
package cn.mrcode.imooc.spring.cloud.apigateway;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

import java.util.Arrays;

/**
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/19 20:49
 */
@Configuration
public class CorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true); // 允许cookie跨域
        config.setAllowedOrigins(Arrays.asList("*"));  // http:www.a.com 原始头
        config.setAllowedHeaders(Arrays.asList("*"));
        config.setAllowedMethods(Arrays.asList("*"));
        source.registerCorsConfiguration("/**", config);
        CorsFilter corsFilter = new CorsFilter(source);
        return corsFilter;
    }
}


```

要了解跨域知识，慕课免费课程挺不错：ajax跨域完全讲解 https://www.imooc.com/learn/947
