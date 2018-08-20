# 服务追踪 上
Spring Cloud Sleuth的使用, Sleuth搭配Zipkin, 直观获取跟踪信息和分析请求链路明细.

* 在什么场景下需要用到服务追踪
* Spring Cloud Sleuth 简介与使用
* Zipkin docker安装
* Sleuth zipkin 如何向zipkin发送链路追踪数据

## 先来看一个场景

```
a     ->      b     ->    c
```
a调用b，b调用c。当有一个需求说，a服务速度有点慢，查清楚是为什么？

传统做法：在a服务中，对调用b，c服务的地方进行分别增加日志，记录请求消耗时间；

这个过程其实就是 链路监控；spring提供了一个解决方案SpringCloudSleuth


## 链路监控 Spring Cloud Sleuth

> https://cloud.spring.io/spring-cloud-sleuth/

> http://cloud.spring.io/spring-cloud-static/spring-cloud-sleuth/2.0.1.RELEASE/single/spring-cloud-sleuth.html

添加依赖
```
compile 'org.springframework.cloud:spring-cloud-starter-sleuth'
```

使用postman来下个订单（订单里面）；观察控制台日志信息
```
POST /order/create HTTP/1.1
Host: localhost:9001
Cache-Control: no-cache
Postman-Token: 567914f3-fef4-4e3e-9c2f-17d759e444ad
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="name"

46465
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="phone"

1850000000
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="address"

xxxxxxxxx
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="openid"

xxx
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="items"

[{"productId":"157875196366160022"}]
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

日志信息
```
2018-08-20 11:41:10.238  INFO [order,308d628cbd6dde2b,308d628cbd6dde2b,false] 14724 --- [nio-9001-exec-6] s.c.a.AnnotationConfigApplicationContext : Refreshing SpringClientFactory-product: startup date [Mon Aug 20 11:41:10 GMT+08:00 2018]; parent: org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@3af37506
2018-08-20 11:41:10.285  INFO [order,308d628cbd6dde2b,308d628cbd6dde2b,false] 14724 --- [nio-9001-exec-6] f.a.AutowiredAnnotationBeanPostProcessor : JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
2018-08-20 11:41:10.568  INFO [order,308d628cbd6dde2b,308d628cbd6dde2b,false] 14724 --- [nio-9001-exec-6] c.netflix.config.ChainedDynamicProperty  : Flipping property: product.ribbon.ActiveConnectionsLimit to use NEXT property: niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit = 2147483647
2018-08-20 11:41:10.594  INFO [order,308d628cbd6dde2b,308d628cbd6dde2b,false] 14724 --- [nio-9001-exec-6] c.n.u.concurrent.ShutdownEnabledTimer    : Shutdown hook installed for: NFLoadBalancer-PingTimer-product
2018-08-20 11:41:10.619  INFO [order,308d628cbd6dde2b,308d628cbd6dde2b,false] 14724 --- [nio-9001-exec-6] c.netflix.loadbalancer.BaseLoadBalancer  : Client: product instantiated a LoadBalancer: DynamicServerListLoadBalancer:{NFLoadBalancer:name=product,current list of Servers=[],Load balancer stats=Zone stats: {},Server stats: []}ServerList:null
2018-08-20 11:41:10.625  INFO [order,308d628cbd6dde2b,308d628cbd6dde2b,false] 14724 --- [nio-9001-exec-6] c.n.l.DynamicServerListLoadBalancer      : Using serverListUpdater PollingServerListUpdater
2018-08-20 11:41:10.656  INFO [order,308d628cbd6dde2b,308d628cbd6dde2b,false] 14724 --- [nio-9001-exec-6] c.netflix.config.ChainedDynamicProperty  : Flipping property: product.ribbon.ActiveConnectionsLimit to use NEXT property: niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit = 2147483647
2018-08-20 11:41:10.658  INFO [order,308d628cbd6dde2b,308d628cbd6dde2b,false] 14724 --- [nio-9001-exec-6] c.n.l.DynamicServerListLoadBalancer      : DynamicServerListLoadBalancer for client product initialized: DynamicServerListLoadBalancer:{NFLoadBalancer:name=product,current list of Servers=[localhost:8080],Load balancer stats=Zone stats: {defaultzone=[Zone:defaultzone;	Instance count:1;	Active connections count: 0;	Circuit breaker tripped count: 0;	Active connections per server: 0.0;]
```

`[order,308d628cbd6dde2b,308d628cbd6dde2b,false]`

> [官网解释](http://cloud.spring.io/spring-cloud-static/spring-cloud-sleuth/2.0.1.RELEASE/single/spring-cloud-sleuth.html#_features)

信息顺序含义是：

* appname：记录范围的应用程序的名称。
* traceId：包含范围的延迟图的ID。
* spanId：发生的特定操作的ID。
* exportable：是否应将日志导出到Zipkin。你想什么时候不能出口？当您想要在Span中包装某些操作并将其写入日志时。

只会在系统启动后，第一次访问服务的时候会打印，后面访问默认不打印日志的。需要开启debug
```yaml
logging:
  level:
    org.springframework.cloud.openfeign: debug
```

那么如果调用的服务也是一个 sleuth 程序，那么traceId将会进行传递。

把product也配置成sleuth程序。再次发送下单请求，就会发现，在order和product中都会出现相同的traceId

那么在控制台查看显示是不太方便的。Zipkin就提供了能在web应用程序中查看的功能

## Zipkin
> https://zipkin.io/

> [官网快速入门](https://zipkin.io/pages/quickstart.html)


使用docker安装 zipkin
```
docker run -d -p 9411:9411 openzipkin/zipkin
```
安装完成后访问：localhost:9411

Zipkin 只是一个ui展示程序，数据从哪里哪里？就要用到  sleuth_with_zipkin

## sleuth zipkin
> [spring官网 zipkin ](http://cloud.spring.io/spring-cloud-static/spring-cloud-sleuth/2.0.1.RELEASE/single/spring-cloud-sleuth.html#_sleuth_with_zipkin_via_http)

> [spring 官网 如何配置 往 zipkin 发送数据](http://cloud.spring.io/spring-cloud-static/spring-cloud-sleuth/2.0.1.RELEASE/single/spring-cloud-sleuth.html#_sending_spans_to_zipkin)

引入依赖
```
compile 'org.springframework.cloud:spring-cloud-starter-zipkin'
```
并配置要发送数据的地址

```yaml
spring:
  zipkin:
    # 默认值是 http://localhost:9411/
    base-url: http://192.168.216.128:9411/
    # 发送方需要写，否则不会发送：支持的有 web，rabbit或kafka
    sender:
      type: web
  sleuth:
    sampler:
      # 抽样观察比例，默认值为0.1，百分比1，开发模式下发送所有的数据
      probability: 1
```

## 测试，输出数据

之前没有敲业务代码，这里指定一下测试步骤：

1. 入口地址： /order/create
2. 该服务中调用：/product/listForOrder
3. 然后调用：/product/list

### 先来看一个产生错误的服务追踪：

前提：
1. 断路器是默认设置，超时时间为1秒
2. 在 /product/list 服务中休眠两秒

这个是访问后的截图展示：

首页列表
![](/assets/image/imooc/spring_secunity/snipaste_20180820_150836.png)

详情，点击某一个报告会进入详情页面

![](/assets/image/imooc/spring_secunity/snipaste_20180820_150911.png)

上图可以看到，

* 整个/order/create服务中调用了2个其他的服务，
* 蓝色的调用成功了，
* 红色的调用失败了 ： 红色接口被调用了三次，这个应该是重试机制？

![](/assets/image/imooc/spring_secunity/snipaste_20180820_151821.png)

上图可以看出，具体的错误信息

### 再来看一个成功的服务追踪

前提：把之前添加休眠的代码去掉。让这个请求成功

![](/assets/image/imooc/spring_secunity/snipaste_20180820_152155.png)

这里就可以看出来，在访问/product/listForOrder的时候花费的实际是最长的。（因为这个接口里面访问了好几次数据库）

我尝试把商品服务也加入zipkin,会发现，同一个链路也会出现在商品服务中，应该是只要涉及到了自己的服务，调用链就会显示出来
