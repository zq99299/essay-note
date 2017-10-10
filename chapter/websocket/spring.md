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


所以本笔记将分为三个demo来实现一些小应用。










# spring-websocket

之前用过spring-websocket写过一个聊天demo。两年时间了。由于工作需要再次接触。记录下这次遇到的坑

```java
后端使用的gradle，在原有的基础上增加spring-websocket
compile "org.springframework:spring-websocket:4.2.3"
```

遵循spring官网文档，尝试；https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-fallback

## 服务端
**websocket配置**
```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        // 这里开启sockjs协议支持
        registry.addHandler(new MyHandler(), "/api/myHandler").setAllowedOrigins("*")
               .addInterceptors(new HttpSessionHandshakeInterceptor())
                .withSockJS();
            
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```

消息处理器
```java
public class MyHandler extends TextWebSocketHandler {
    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message) {
        System.out.println(message);
    }

}
```


## 前端
由于使用的vue，只能在npm中安装`"sockjs-client": "^1.1.4"`

使用:
```javascript
import SockJS from 'sockjs-client'

var sock = new SockJS('/api/myHandler')
sock.onopen = function () {
    console.log('open')
    sock.send('test')
}

sock.onmessage = function (e) {
    console.log('message', e.data)
    sock.close()
}

sock.onclose = function () {
    console.log('close')
}
```

## 开始测试-5个小时的错误解决

### 主要错误解决
打开谷歌浏览器的控制台，查看会有一堆的错误提示出现，最开始的是下面的错误：
```
EventSource's response has a MIME type ("application/json") that is not "text/event-stream". Aborting the connection.
sockjs.min.js:2 Uncaught Error: Incompatibile SockJS! Main site uses: "1.1.4", the iframe: "1.0.0".
```

主主要的错误应该就是说 服务器端使用的版本是1.0.0,而js中使用的是1.1.4; 解决这个问题其实在[官网文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-fallback-xhr-vs-iframe)中已经写到了

解决方案：在配置类中配置和前端js使用相同版本的js
```java
public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
    // 这里开启sockjs协议支持
    registry.addHandler(new MyHandler(), "/api/myHandler").setAllowedOrigins("*")
    .addInterceptors(new HttpSessionHandshakeInterceptor())
    .withSockJS().setClientLibraryUrl("//cdn.jsdelivr.net/sockjs/1/sockjs.min.js");
}
```
上面的cdn指向的地址就是 [sockjs官网](https://github.com/sockjs/sockjs-client) 起步实例中的cdn文件。

**服务器端的错误则是：**
```java
Caused by: java.lang.IllegalArgumentException: Async support must be enabled on a servlet and for all filters involved in async request processing. This is done in Java code using the Servlet API or by adding "<async-supported>true</async-supported>" to servlet and filter declarations in web.xml. Also you must use a Servlet 3.0+ container
```

看这里要开启异步支持。由于我这里使用的是web.xml 所以在web.xml中开启,

这个在[官网文档中提到过要开启异步支持](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-fallback-sockjs-servlet3-async)

在网上查了下 web.xml中开启方法如下;
```xml
    <filter>
        <filter-name>httpPutFormFilter</filter-name>
        <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
        <async-supported>true</async-supported>
    </filter>
     <servlet>
        <servlet-name>mvcDispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
        <async-supported>true</async-supported>
    </servlet>
```

只要有 `<filter>`的地方都要添加`<async-supported>true</async-supported>`,还有就是mvcDispatcher的入口处

### 失败：连接在收到握手响应之前关闭
最主要的是上面两个错误，解决完成之后就可以链接上并也能前端往后端发送信息了。在前端控制台中，又冒出来一个如下的错误：
```javascript
websocket.js?0f24:6 WebSocket connection to 'ws://localhost:9105/api/myHandler/761/czzeyw3z/websocket' failed: Connection closed before receiving a handshake response
```

出现这个错误后，但是能正常的通信；只是在打印open之前报错了。后来找到了问题；

我使用的是vue-cli开发，用了代理转发到后台，所以在链接的时候使用了前端的地址，让代理转发的。代码如下
```javascript
var sock = new SockJS('/api/myHandler')
```

把相对路径更改为后端服务的直接路径，成功链接上
```javascript
var sock = new SockJS('http://localhost:9104/api/myHandler')
```

## 几个概念

1. `WebSocketSession` 有几个？

    在spring中通过测试，addHandler对应的地址相同的在一个session周期中都是同一个sessionId.




