# 课程介绍

## spring boot 和 spring cloud的关系
```
↑ SringBoot
↑ SpringCloud
↑ 微服务
```

```
↑ SpringCloud
↑ SringBoot
↑ Spring Framework
```
SpringBoot对spring框架的一个简化，快速构建spring应用；
而springCloud又是对SpringBoot简化快速构建分布式应用的框架

## 技术储备

1. 对spring boot 的基础只是熟练掌握
2. 对Linux和和Docker的基本用法熟练掌握

## 课程重点
1. spring cloud构建微服务 ； 从零构建
2. 微服务改造探讨

## 为什么使用SpringBoot2.x?
截止本笔记正式版已经发布（18年2月27号）

spring cloud子项目非常多，重点介绍以下几个：

* Eureka 服务注册与发现
  - Eureka Server
  - Eureka Client
  - Eureka 高可用
  - 服务发现机制

* Config 配置
  - Config Server
  - Config Client
  - Spring Cloud Bus 结合RabibtMQ 实现配置自动刷新

* Ribbon 通信
  - RestTemplate
  - Feign
  - Ribbon
  - 分析源码了解底层

* zuul 网关
  - 动态路由
  - 校验

* Hystrix 熔断
  - Hystrix Dashboar
  - 熔断机制

* 容器编排 ： docker + rancher
* 服务追踪 ： spring cloud sleuth + zipkin

## 源码/资源获取
常见问题汇总
2018-04-22
https://gitlab-demo.com/SpringCloud_Sell/doc/blob/master/QA.md

虚拟机下载
2018-04-07
链接:https://pan.baidu.com/s/1tWje5AK68dZ887OylBHg9g 密码:723y 账号 root 密码 123456

SpringCloud源码获取
2018-03-05
源代码获取说明请看：https://www.imooc.com/article/23867
