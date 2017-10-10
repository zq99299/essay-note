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