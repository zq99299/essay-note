# websocke-demo

后端的构建就不说了。直接开始说相关的。具体的项目我会放在git上。

在仓库中找到 [spring-websocket](http://mvnrepository.com/artifact/org.springframework/spring-websocket/4.2.3.RELEASE)

在依赖项中可以看到依赖的具体包。这里直接引入这一个即可

```
compile "org.springframework:spring-websocket:${springframeworkVersion}"
```

接下来就开始写了。
## 创建并配置WebSocketHandler

WebSocketHandler的作用是，当有消息进入的时候，你要怎么处理？

spring提供给我们两种可用接口 TextWebSocketHandler或BinaryWebSocketHandler：
```java
public class MyHandler extends TextWebSocketHandler {
    private Logger log = LoggerFactory.getLogger(getClass());

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        log.info("收到消息：sessionId={},msg={}", session.getId(), message);
    }
}
```

上面的处理器有专门的配置映射到一个url上。
```java
@Configuration
@EnableWebSocket  // 开启websocket支持
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        // 添加处理器
        registry.addHandler(myHandler(), "/myHandler");
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }
}

```

启动项目，访问`http://localhost:8080/myHandler`,如果没有问题的话，你会看到`Can "Upgrade" only to "WebSocket".`字样的输出。

这个是什么意思呢？前面说到过
> HTTP仅用于初始握手，这取决于内置于HTTP中的机制来请求协议升级（或在这种情况下为协议交换机），服务器可以使用HTTP状态101对其进行响应（切换协议）如果它同意。假设握手成功，HTTP升级请求下面的TCP套接字保持打开，客户端和服务器都可以使用它来彼此发送消息。