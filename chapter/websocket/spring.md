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

### 第一个错误
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




