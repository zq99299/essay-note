# stomp-点对点-自由订阅

所谓点对点，具体的源码我没有去看过，在spring-websocket 3.4.5 以前的版本我无意中在网络上的各种资料中尝试了很多写法。只有一种让我成功了。

```java
前端订阅地址写：/topic/用户名/xxx
后端发送的时候也是:/topic/用户名/xxx
```
这样看来其实都是走的消息代理，只是这个订阅地址，只有被授权的人知道罢了。 大概的跟着了下源码。stomp的大致原理也是这样的。 

拦截user开头的前缀然后拼接用户名`/user/用户名/具体的订阅地址`,后端发送的时候，调用的api也是专用的，该api也会把地址拼接成拦截时候修改的那样。完成了一个转换。更具体的原理没有去深挖。


好了基本的了解之后。来实现点对点订阅

其实对于api来说不算什么难事。我反倒是对于怎么维护状态比较关心。

平时的http一个请求一个响应，是无状态的。现在使用同一个链接，不同地址路由到不同的方法中去，这个也是没有问题的，要实现后端主动推送变化的数据，你还得保存每个用户所订阅类别是什么。这就变成了有状态。那么总结出来有以下几个点需要解决：

1. 怎么保存用户的选择状态
2. 状态保存后，什么时机清空该状态？（用户断开了stomp链接、离开了该页面等）

## 前端实现

考虑一下该需求。这里是两个不同的新闻类型，可以看成是 查询条件。只要传递不同的条件则返回的数据不一样。

这里我把两个订阅的合并成一个来实现，是为了演示怎么使用发送消息来改变条件，和服务器交互。

前端怎么通过stomp发送条件到后端呢？

发送信息很简单，提供一个消息地址，提供发送的内容即可。
```javascript
      comicSubscribe () {
        // 发送信息
        this.varStore.stomp.send('/app/queue/other', {}, JSON.stringify({
          type: 1, // 订阅
          body: 'comic' // 订阅的内容是动漫
        }))
      }
```

这里的地址要说明下，前面提到过，app前缀开头的会被路由到应用程序中，什么算应用程序呢？在springmvc中（Controller），被以下注解的方法，都会被路由

1. `@MessageMapping` : 该注解类似`@RequestMapping`,提供一个地址，只要访问这个地址，就会被映射到该方法中
2. `@SubscribeMapping` : 该注解也是提供一个地址，把对应的请求映射到此方法，只不过该注解 的效果是，订阅一次该地址，则消息立马返回到订阅的客户端，不会走消息代理。而且只会返回一次。所以该注解可以用来做一些订阅的时候拉取初始化信息的功能，获取初始化信息的场景。


所以后端需要使用 `@MessageMapping` 来处理提交的信息

## 使用 `@MessageMapping`处理提交的条件信息

```java

    @MessageMapping("/queue/other")
    public void otherSubscribe(Command command) {
        System.out.println(command);
    }
```

测试发现报错了

```java
 org.springframework.messaging.converter.MessageConversionException: No converter found to convert to class cn.mrcode.javawebsocketdemo.stomp.controller.Command, message=GenericMessage [payload=byte[25], headers={simpMessageType=MESSAGE, stompCommand=SEND, nativeHeaders={destination=[/app/queue/other], content-length=[25]}, simpSessionAttributes={}, simpHeartbeat=[J@6a00f3b2, lookupDestination=/queue/other, simpSessionId=hkgcxx3t, simpDestination=/app/queue/other}]
	at org.springframework.messaging.handler.annotation.support.PayloadArgumentResolver.resolveArgument(PayloadArgumentResolver.java:118)
	at org.springframework.messaging.handler.invocation.HandlerMethodArgumentResolverComposite.resolveArgument(HandlerMethodArgumentResolverComposite.java:77)
	at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.getMethodArgumentValues(InvocableHandlerMethod.java:139)
```

错误原因： 发送消息的时候，消息解析会走这个方法，`org.springframework.messaging.converter.CompositeMessageConverter#fromMessage(org.springframework.messaging.Message<?>, java.lang.Class<?>, java.lang.Object)`里面有一些消息转换器，默认的有 `StringMessageConverter` 和 `ByteArrayMessageConverter` ; 那么这两个都不能解析我们发送的json字符串，他是根据你要接收信息的类型来判定的，如果是字符串，那么可能就成功了。但是这里后端提供的是一个对象。

所以去仓库看了下依赖：
`com.fasterxml.jackson.core » jackson-databind (optional)	2.8.5	2.9.2` ;[jackson是可选的](http://mvnrepository.com/artifact/org.springframework/spring-messaging/4.3.5.RELEASE)。 之前项目中也没有爆出这个错误，我同样跟踪了源码，发现多了一个转换器`MappingJackson2MessageConverter`； 这里就很明白了。 或许就是这个jackson包的问题。我又没有发现怎么自定义添加转换器，不得已，只能先加上这个包了；

再次调试，发现果然多了一个转换器。该问题成功解决。




-- 待续
