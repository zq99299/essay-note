# 服务追踪 下

集成步骤总结：
1. 引入先关依赖
2. 启动 zipkin server
3. 配置参数

## 理论知识

核心步骤

* 数据采集 ：
* 数据存储格式
* 查询展示

不同系统的数据格式不同，因此产生了[OpenTracing][4c6ee856]协议

  [4c6ee856]: https://blog.csdn.net/u013970991/article/details/77822060 "csdn一篇文章"

## OpenTracing
优势

* 来自大名鼎鼎的CNCF
* ZIPKIN,TRACER,JAEGER,GRPC等都使用了该协议

事件类型：

* cs：Client Send : 客户端发起请求的时间
* sr：Server Received ：服务端收到调用请求的时间
* ss：Server Send : 服务端处理完逻辑的时间
* cr：Client Received ：客户端收到处理完请求的时间

上面的术语也在 [spring-cloud-sleuth][e9b4f7cf] 中有说明

  [e9b4f7cf]: http://cloud.spring.io/spring-cloud-static/spring-cloud-sleuth/2.0.1.RELEASE/single/spring-cloud-sleuth.html#_terminology "spring-cloud-sleuth 术语解释"

* 客户端调用时间 = cr - cs
* 服务端处理时间 = sr - ss

## zipKin

Zipkin的设计基于 [Google Dapper](https://ai.google/research/pubs/pub36356) 论文。

twitter 根据 该论文开发出了 追踪系统，并开源

## 几个关键概念

* traceId : 一个调用链id
* spanId ：调用服务id
* parentId ：父id，把整个联调串联起来的映射
