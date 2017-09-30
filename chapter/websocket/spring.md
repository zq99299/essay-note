# spring-websocket

之前用过spring-websocket写过一个聊天demo。两年时间了。由于工作需要再次接触。记录下这次遇到的坑

```java
后端使用的gradle，在原有的基础上增加spring-websocket
compile "org.springframework:spring-websocket:4.2.3"
```

遵循spring官网文档，尝试；https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-fallback

**websocket配置**
```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        HandshakeInterceptor handshakeInterceptor = new HandshakeInterceptor();
        registry.addHandler(new MyHandler(), "/api/echo"); //
        registry.addHandler(new MyHandler(), "/api/myHandler").setAllowedOrigins("*")
//                .addInterceptors(new HttpSessionHandshakeInterceptor())
//                .addInterceptors(handshakeInterceptor)
                .withSockJS()
                .setClientLibraryUrl("//cdn.jsdelivr.net/sockjs/1/sockjs.min.js");
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

