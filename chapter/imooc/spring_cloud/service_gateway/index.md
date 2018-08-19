# 服务网关

## 服务网关和Zuul

为什么需要网关服务？ 作为所有服务的入口

服务网关的要素：

* 稳定性，高可用
* 性能，并发性
* 安全性
* 扩展性

理论上网关上处理非业务功能。

## 常用的网关方案

* Nginx + Lua  : nginx性能极高
* Kong   : 基于nginx，商业软件
* Tyk  ： go语言开发，支持的也挺多
* SpringCloudZuul ：快速上手

## Zuul的特点

* 路由+过滤器=Zuul
* 核心是一系列的过滤器

zuul的四中过滤器

* 前置（pre）
* 后置（post）
* 路由（route）
* 错误（error）

结构图：
![](/assets/image/imooc/spring_cloud/snipaste_20180819_152315.png)

生命周期
![](/assets/image/imooc/spring_cloud/snipaste_20180819_152345.png)

红色的是过滤器，是一系列实现类；

## zuul体验
> zuul 官网文档
>
> http://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.0.1.RELEASE/single/spring-cloud-netflix.html#_router_and_filter_zuul


```
Cloud Discover -> Eureka Discover
Cloud Routing -> Zuul
```

生成的依赖
```
dependencies {
	compile('org.springframework.cloud:spring-cloud-starter-netflix-eureka-client')
	compile('org.springframework.cloud:spring-cloud-starter-netflix-zuul')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

yml配置文件,其实没有和zuul相关的配置；
```yaml
server:
  port: 9900
spring:
  application:
    name: api-gateway

eureka:
  instance:
    hostname: localhost
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

开启zuul
```java
package cn.mrcode.imooc.spring.cloud.apigateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableEurekaClient
@EnableZuulProxy
public class ApiGatewayApplication {

	public static void main(String[] args) {
		SpringApplication.run(ApiGatewayApplication.class, args);
	}
}

```

原始的商品服务列表地址：http://localhost:8080/product/list
使用zuul来访问：http://localhost:9900/product/product/list

第一个product是商品服务的服务id；zuul的默认格式就是以服务id进行转发

## 自定义路径转发

配置规则，拦截所有 /myProduct的路径，然后转发到product服务上
```
zuul:
  routes:
    myProduct:
      path: /myProduct/**
      serviceId: product
    // 下面这种方式等价于上面的配置  
    product: /myProduct2/**
```
访问地址： http://localhost:9900//myProduct/product/list

上面配置的规则是一个 ZuulRoute 对象。可以查看可以配置哪些属性
```java
org.springframework.cloud.netflix.zuul.filters.ZuulProperties.ZuulRoute
```

## 禁止路由
`ignored-patterns`属性，可以配置多个或则正则？
```yaml
zuul:
  routes:
    myProduct:
      path: /myProduct/**
      serviceId: product
    product: /myProduct2/**
  ignored-patterns:
    - /**/product/listForOrder
```

## 敏感头 - cookie
> http://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.0.1.RELEASE/single/spring-cloud-netflix.html#_cookies_and_sensitive_headers

`sensitiveHeaders：`属性

默认是忽略了cookie，在上面的属性中，有三个默认值，其中就是cookie

## 动态路由配置

可以通过配置中心来刷新配置信息；

下面这个配置没有尝试过。不知道是否有效
```
@ConfigurationProperties("zuul")
@RefreshScope
public ZuulProperties zuulProperties() {
    return new ZuulProperties();
}
```

## 路由和高可用小结

典型的应用场景：

* 前置（Pre）
  - 限流 ：当峰值达到一定程度的时候，就把请求挡回去了
  - 鉴权 ：
  - 参数校验

* 后置（Post）
  - 统计
  - 日志

## zuul高可用

* 多个Zuul节点注册在EurekaServer
* Nginx和Zuul“混搭” ；对外暴露来说使用nginx转发到多个zuul节点
