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

##

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




