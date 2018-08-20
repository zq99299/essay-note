Spring Cloud微服务实战

* [第1章 课程介绍](/chapter/imooc/spring_cloud/introduce.md)
* 第2章 微服务介绍

  什么是微服务, 单体架构优缺点, 常见的几种架构模式。
  - [微服务和其他常见架构](/chapter/imooc/spring_cloud/microservice/微服务和其他常见架构.md)
  - [从一个极简的微服务架构开始](/chapter/imooc/spring_cloud/microservice/从一个极简的微服务架构开始.md)
* 第3章 服务注册与发现

  介绍微服务中的服务注册与发现机制，Spring Cloud Eureka组件的使用以及如何保证高可用
  - [Spring Cloud Eureka](/chapter/imooc/spring_cloud/register_discovery/spring_cloud_eureka.md)
  - [Eureka Server](/chapter/imooc/spring_cloud/register_discovery/eureka_server.md)
  - [Eureka Client的使用](/chapter/imooc/spring_cloud/register_discovery/eureka_client.md)
  - [Eureka的高可用](/chapter/imooc/spring_cloud/register_discovery/eureka_high_availability.md)
  - [Eureka总结](/chapter/imooc/spring_cloud/register_discovery/eureka_summarize.md)
  - [分布式下服务注册的地位和原理](/chapter/imooc/spring_cloud/register_discovery/分布式下服务注册的地位和原理.md)

* 第4章 服务拆分

  以商品服务和订单服务为例介绍微服务拆分中的业务功能拆分和数据拆分的注意点以及将项目模块进行多模块改造
  - [微服务拆分的起点](/chapter/imooc/spring_cloud/service_split/微服务拆分的起点.md)
  - [康威定律和微服务](/chapter/imooc/spring_cloud/service_split/康威定律和微服务.md)
  - [点餐业务服务拆分分析](/chapter/imooc/spring_cloud/service_split/点餐业务服务拆分分析.md)
  - [商品服务API和SQL介绍](/chapter/imooc/spring_cloud/service_split/商品服务API和SQL介绍.md)
  - [商品服务编码(上)](/chapter/imooc/spring_cloud/service_split/商品服务编码上.md)
  - [商品服务编码(下)](/chapter/imooc/spring_cloud/service_split/商品服务编码下.md)
  - [订单服务API和SQL介绍](/chapter/imooc/spring_cloud/service_split/订单服务API和SQL介绍.md)
  - [订单服务接口实现](/chapter/imooc/spring_cloud/service_split/订单服务接口实现.md)
  - [再看“拆数据”](/chapter/imooc/spring_cloud/service_split/再看“拆数据”.md)

* 第5章 应用通信

  比较HTTP REST 和 REST，同步和异步, 介绍Spirng Cloud 采用的两种HTTP方式，重点介绍Feign. 实例演示下单流程. 引出异步通信的思考.
  - [应用间通信-框架与基本使用](/chapter/imooc/spring_cloud/communication/index.md)
  - [应用间通信-下](/chapter/imooc/spring_cloud/communication/应用间通信-下.md)
* 第6章 统一配置中心

  介绍Spring Cloud Config组件搭配Spring Cloud Bus, 实现配置自动更新, 集成WebHook
  - [config-server的使用](/chapter/imooc/spring_cloud/config_center/config_server.md)
  - [config-client的使用](/chapter/imooc/spring_cloud/config_center/config_client.md)
  - [SpringCoudBus](/chapter/imooc/spring_cloud/config_center/bus.md)
* 第7章 异步和消息

  RabbitMQ，Spring Cloud Stream组件介绍及使用, 异步通信实例演示和思考
  - [rabbitMQ的基本使用](/chapter/imooc/spring_cloud/asyn_msg/rabbitMQ.md)
  - [SpringCloudStream](/chapter/imooc/spring_cloud/asyn_msg/spring_cloud_stream.md)
  - [业务中使用](/chapter/imooc/spring_cloud/asyn_msg/业务中使用.md)
* 第8章 服务网关

  探讨微服务架构下的服务网关，介绍Spring Cloud Zuul的使用, 路由转发, Cookie处理, 动态路由等Zuul路由相关的功能，也探讨了Zuul的高可用
  - [服务网关](/chapter/imooc/spring_cloud/service_gateway/index.md)
* 第9章 Zuul综合使用

  围绕过滤器，选取限流，跨域等典型场景，综合使用Zuul，集成用户服务
  - [pre、post、使用和过滤器限流](/chapter/imooc/spring_cloud/zuul_synthesis/index.md)
  - [业务使用](/chapter/imooc/spring_cloud/zuul_synthesis/temp.md)
* 第10章 服务容错

  探讨熔断机制，Spring Cloud Hystrix的使用, Feign+Hystrix服务降级.
  - [服务容错上 ](/chapter/imooc/spring_cloud/service_fault_tolerance/circuit_breaker.md)
  - [服务容错下 ](/chapter/imooc/spring_cloud/service_fault_tolerance/service_fault_tolerance2.md)
* 第11章 服务追踪

  Spring Cloud Sleuth的使用, Sleuth搭配Zipkin, 直观获取跟踪信息和分析请求链路明细.
  - [服务追踪 上](/chapter/imooc/spring_cloud/service_tracking/inde1.md)
  - [服务追踪 下](/chapter/imooc/spring_cloud/service_tracking/inde2.md)
* 第12章 容器部署

  使用Docker容器+Rancher容器管理平台部署微服务, 资源弹性分配, 容器编排与调度.

  - [运行第一个docker容器](/chapter/imooc/spring_cloud/container_deployment/first.md)
  * 以下不是视频内容
  * [容器相关只是](/chapter/container/index.md)
