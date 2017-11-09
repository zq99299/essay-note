# spring-websocket


## 介绍
WebSocket[协议RFC 6455](https://tools.ietf.org/html/rfc6455)为Web应用程序定义了一个重要的新功能：客户端和服务器之间的全双工，双向通信。

简单来说，他会使用HTTP用于初始握手，这取决于内置于HTTP中的机制来请求协议升级（或在这种情况下为协议交换机），服务器可以使用HTTP状态101对其进行响应（切换协议）如果它同意。假设握手成功，HTTP升级请求下面的TCP套接字保持打开，客户端和服务器都可以使用它来彼此发送消息。


Spring Framework 4包含一个全面的WebSocket支持模块spring-websocket。它与Java WebSocket API标准（[JSR-356](https://jcp.org/en/jsr/detail?id=356)）兼容，并且还提供了额外的附加值

## 什么时候适合使用websocket?

WebSocket最适合Web应用程序，客户端和服务器需要以高频率和低延迟交换事件。协同，游戏，聊天，金融股票展示等类型的应用，这样的应用对时间延迟非常敏感，并且还需要以高频率交换各种各样的消息。

低延迟和高频率的消息的组合可以使WebSocket协议的使用成为关键。

Spring框架允许@Controller和@RestController类具有HTTP请求处理和WebSocket消息处理方法。此外，Spring MVC请求处理方法或任何应用方法可以轻松地向所有感兴趣的WebSocket客户端或特定用户广播消息。

## websocket、sockJS、STOMP 之间的区别


### websocket
WebSocket 可以看成是消息架构，但不要求使用任何特定的消息传递协议。它是一个非常薄的TCP层，将字节流转换为消息流（文本或二进制），由应用来解释消息的含义。与HTTP（它是一个应用程序级协议）不同，在WebSocket协议中，框架或容器的传入消息中没有足够的信息来知道如何路由或处理它。因此，WebSocket可以说是太低级别，只是一个非常简单的应用程序。

### sockJS
是一个协议，是websocket的备胎，一些浏览器不支持websocket的话，该协议则使用原始的ajax轮询等手段来模仿支持websocket的行为。

### STOMP 
 STOMP是一种简单的消息协议，最初创建用于脚本语言，具有灵感来自HTTP的框架。STOMP得到广泛的支持，非常适合在WebSocket和Web上使用。
 
 简单说就是STOMP适合封装在websocket上面。提供了消息路由的功能（给我最大的感受），它的使用类似与Java中的消息队列，发布订阅模型。基于地址进行消息的路由。


## 本笔记的干货介绍

上面的介绍是困扰了我很久的东西，更详细的没有时间和精力去研究。简单来看差不多是ok了。

所以本笔记将会有3个demo进行讲解spring websocket的使用。

* 低级的spring-websocket使用
* 基于sockJS的使用
* 重点-基于STOMP的使用

### 后端项目介绍
不会使用spring-boot，因为我不太会，使用原始的web.xml 的方式。

gradle 构建项目；包含子模块websock 和 stomp

### 前端项目介绍
前端使用 vue-cli 来构建，一个项目，分为3个页面。分别来讲解低级的，sockJS,stomp 怎么与后端使用。

由于前面的比较简单，着重讲解我对stomp的一些理解。






