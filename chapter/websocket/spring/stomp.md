# stompDemo

由于stomp的配置和使用都和原生的不太一样了。所以这里重新开一个项目stomp； 基本项目结构使用前面的。

## 后端初体验

stomp的配置和之前的有点不太一样。

`@EnableWebSocketMessageBroker` : 开启stomp，在使用过程中发现的是，该注解会开启几个线程池，然后使用线程池来对消息的监控发送等功能，所以，如果在项目中有报错的话。注意查看 是否有相关的调度任务等。


```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // 增加一个入口节点，并开其sockjs的支持，和相关的配置
        registry.addEndpoint("/stomp").setAllowedOrigins("*")
                .withSockJS()
                .setMessageCodec(new FastjsonSockJsMessageCodec())
        ;
    }
}
```

启动项目后，发现后台报错了：

```java
org.springframework.beans.factory.BeanDefinitionStoreException: Failed to process import candidates for configuration class [cn.mrcode.javawebsocketdemo.stomp.ws.WebSocketConfig]; nested exception is java.io.FileNotFoundException: class path resource [org/springframework/messaging/simp/config/AbstractMessageBrokerConfiguration.class] cannot be opened because it does not exist
```

这里用到了messaging的包。可以看出来stomp是通过messaging包扩展出来的；所以我们把messaging包增加进来

```java
compile "org.springframework:spring-messaging:${springframeworkVersion}"
```

再次启动，发现报错解除；这个时候来写前端页面

## 前端初体验

Stomp库官网文档：http://jmesnil.net/stomp-websocket/doc/

直接按照文档的几个简单的api来


