# stomp-广播-公共新闻订阅

之前的配置和代码已经满足不了现在的需求了。先来看看后端的新配置

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/stomp").setAllowedOrigins("*")
                .withSockJS()
                .setMessageCodec(new FastjsonSockJsMessageCodec())
//                .setClientLibraryUrl("//cdn.jsdelivr.net/sockjs/1/sockjs.min.js");
        ;
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        // 前端发送消息和主动访问对应的消息地址前缀
        config.setApplicationDestinationPrefixes("/app");
        // 开启消息代理
        config.enableSimpleBroker("/topic", "/queue");
        // 点对点支持前缀
        config.setUserDestinationPrefix("/user/");
    }

}
```

这里新增了`configureMessageBroker`配置。

* `config.setApplicationDestinationPrefixes("/app")` 
    
    以`/app` 前缀开头的消息请求 将被路由到 应用程序 Controller 中的处理方法,被消息相关注解声明的方法中，后面会讲到
    
* `config.enableSimpleBroker("/topic", "/queue");`
    
    这里注册了两个路径，这两个路径没有什么分别，只是语义化的区分。topic 作用广播前缀，queue作为点对点的前缀. 以这些前缀开始的路径将会被路由到消息代理中去。
    
    [这里是官网的示意图，可以去看一下](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-message-flow)，如果现在看不懂，不要紧，本节讲完，大概就能明白了 
    
* `config.setUserDestinationPrefix("/user/")`;

    以user开头的将会被框架代理，实际产生的路径不是我们写的那样。这个后面也会讲
    

配置好了。还需要一个一个线程来定时发送数据。用来模拟有新的新闻出现


为了方便，这里我们把模拟线程直接写在 DemoController 中了。
```java
@RestController
@RequestMapping(produces = BaseController.CHARSET_UTF8_JSON)
public class DemoController extends BaseController {
    @Autowired
    private SimpMessagingTemplate template;

    {
        new Thread(() -> {
            mockPublicNews();
        }).start();
    }

    /**
     * 模拟公共新闻推送
     */
    public void mockPublicNews() {
        while (true) {
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            List<String> list = new ArrayList<>();

            // 模拟产生3条
            list.add("清华大学16位学霸为一事现身PK 简历吓坏网友 - " + randomInt());
            list.add("闹市区树上长出6斤多蘑菇 保安大叔炒着吃了(图) - " + randomInt());
            list.add("女司机第2次上路撞死过路老人 教练被抓走 司法频道 - " + randomInt());


            // 广播消息，注意记住这里的地址
            template.convertAndSend("/topic/public_news", JSON.toJSON(list));
        }
    }

    private int randomInt() {
        return RandomUtils.nextInt(100, 1000);
    }
}
```


    