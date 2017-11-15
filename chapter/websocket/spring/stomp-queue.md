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


ok。这里可以接受消息了。那么来处理真正的需求

## 授权认证

点对点前面已经说到过了，需要一个用户名，或则一个能表示该用户的一个标识。那么这个标识怎么来呢？ 就是在链接的时候进行认证授权


还记得在js中链接的时候 那个头吗。这里可以传递用户名什么的。 我处理的逻辑是用户登录后，然后再链接，这个时候已经拿到用户名或则用户Id了。直接传递一个login就行了。这里固定的写死。
```javascript
        let headers = {
          login: 'mylogin',
          passcode: 'mypasscode',
          // additional header
          'client-id': 'my-client-id'
        }
```

后端要做的是
在websocketconfig这里面注册拦截器

```java
    /**
     * 以下是注册自定义身份验证拦截器的示例服务器端配置。
     * 请注意，拦截器只需要认证并设置CONNECT上的用户头Message。
     * Spring将注意并保存已验证的用户，并将其与后续STOMP消息相关联在一起：
     * @param registration
     */
    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.setInterceptors(new ChannelInterceptorAdapter() {

            @Override
            public Message<?> preSend(Message<?> message, MessageChannel channel) {
                StompHeaderAccessor accessor =
                        MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);


                // 只要链接一次。后面每次都是根据类型走，而且用户都会存在
                StompCommand command = accessor.getCommand();
                if (StompCommand.CONNECT.equals(command)) {
                    MyPrincipal principal = new MyPrincipal(accessor.getLogin());
                    accessor.setUser(principal);
                }
                log.info("请求用户:{}", accessor.getUser());
                return message;
            }
        });
    }
```

在Controller方法声明中添加参数注入

```java
    @MessageMapping("/queue/other")
    public void otherSubscribe(@Payload Command command, Principal principal, SimpMessageHeaderAccessor headerAccessor) {
        int type = command.getType();
        if (type == 1) { // 订阅

        }
        System.out.println(command);
        System.out.println(principal);
    }
```

* `Principal` : 这里能获取在上面拦截器中我们自己setUser的对象
* `SimpMessageHeaderAccessor` : 更详情的一些信息，包括 sessionId,消息id等
* `@Payload` 注解表示你要用来接收消息内容的对象是哪一个


到此位置，授权认证已经通过。我们也能拿到用户的信息了。


## 完善订阅后端逻辑

后端也需要开启一个线程来模拟有新的新闻出现，还要维护用户订阅的参数信息

```java
 /**
     * 记住这里的地址
     */
    @MessageMapping("/queue/other")
    public void otherSubscribe(@Payload Command command, Principal principal, SimpMessageHeaderAccessor headerAccessor) {
        int type = command.getType();
        if (type == 1) { // 订阅
            subscribeNews(principal.getName(), command.getBody());
        }
        System.out.println(command);
        System.out.println(principal);
    }

    private Map<String, SubscribeParams> subscribeMap = new HashMap<>();

    /**
     * 订阅线程处理
     */
    private void mockSubscribeNews() {
        while (true) {
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            subscribeMap.forEach((user, params) -> {
                JSONObject result = new JSONObject();
                if (params.isComic()) {
                    result.put("comics", buildComic());
                }
                if (params.isGossip()) {
                    result.put("gossips", buildGossip());
                }
                template.convertAndSendToUser(user, "/queue/other", result.toJSONString());
            });
        }
    }

    private List<String> buildComic() {
        List<String> list = new ArrayList<>();
        list.add("久保带人大爆八卦内幕：「死神」差点拍好莱坞版？想休息15年不画新作品？ - " + randomInt());
        list.add("1月霸权番预定？「DARLING in the FRANXX」首曝PV，户松遥等参演！ - " + randomInt());
        return list;
    }

    private List<String> buildGossip() {
        List<String> list = new ArrayList<>();
        // 模拟产生3条
        list.add("言承旭自卑感作祟感情路不顺 羡慕周杰伦完美成家 - " + randomInt());
        list.add("贺涵老卓再合作？靳东晒和陈道明自拍笑出一脸褶 - " + randomInt());
        list.add("林心如被娶走改追回林志玲？言承旭经纪人打脸回应 - " + randomInt());
        return list;
    }


    /**
     * 接受处理参数
     * @param user     用户名
     * @param newsType 订阅的新闻类别是什么
     */
    private void subscribeNews(String user, String newsType) {
        SubscribeParams subscribeParams = null;
        if (subscribeMap.containsKey(user)) {
            subscribeParams = subscribeMap.get(user);
        } else {
            subscribeParams = new SubscribeParams();
            subscribeMap.put(user, subscribeParams);
        }

        if ("comic".equals(newsType)) {
            subscribeParams.setComic(true);
        } else {
            subscribeParams.setGossip(true);
        }
    }
```

这样后端完成了，前端需要订阅这个地址，才行。改造前端代码

```javascript
 // 动漫订阅
      comicSubscribe () {
        this.otherSubscribe('comic')
        this.otherSubscribeInit()
      },
      otherSubscribeInit () {
        // 如果还没有开启一个订阅实例，则开启
        if (!this.varStore.otherSubscribe) {
          // 注意这里的地址；和后端发送的地址是一样的;只是增加了/user的前端，该前缀标识是一个点对点的订阅
          // 后端框架会特殊处理
          this.varStore.otherSubscribe = this.varStore.stomp.subscribe('/user/queue/other', message => {
            let news = JSON.parse(message.body)
            // 把获取到的列表赋值给该变量，页面中会循环出该信息
            this.subscribeNews.comics = news.comics
            this.subscribeNews.gossips = news.gossips
          })
        }
      },
      otherSubscribe (type) {
        // 发送信息
        this.varStore.stomp.send('/app/queue/other', {}, JSON.stringify({
          type: 1, // 订阅
          body: type // 订阅的内容是动漫还是八卦类型
        }))
        if (type === 'comic') {
          this.subscribeNews.comic = true
        } else {
          this.subscribeNews.gossip = true
        }
      }
    }
```

ok，点对点的应用结束，后面的我就不记录了。直接在程序中写完这个demo。并放在git上。

## 总结：

1. 点对点发送，必须要在spring4.3.5+版本。才有效，才会有效的帮你转换订阅地址
2. 订阅的地址是走代理的。前后都必须是同一个地址才会收到对应的消息。
3. 由于框架的规则，前后写的时候 附带一些前缀，在视觉上除了前缀之外，其他的地址应该相同，对第二条的补充

websocket的知识还是有点多的感觉。使用上也是不同的。 这个只是我个人摸索得出来的一种使用方式。更多的还是去查看官网文档。这里带你入门


