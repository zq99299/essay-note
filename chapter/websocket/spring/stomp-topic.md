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
            template.convertAndSend("/topic/public_news", JSON.toJSONString(list));
        }
    }

    private int randomInt() {
        return RandomUtils.nextInt(100, 1000);
    }
}
```

后端服务写好后来实现前端的逻辑。
## 前端订阅

在之前的代码基础上，链接成功后，调用该方法；
```javascript
topicPublicNews () {
        // 注意这里的地址，和 后端发送消息的地址是一致的
        this.varStore.stomp.subscribe('/topic/public_news', message => {
          let news = JSON.parse(message.body)
          // 把获取到的列表赋值给该变量，页面中会循环出该信息
          this.publicNews = news
        })
      }
```

打开页面测试该功能，并查看浏览器控制台打印出来的信息如下：

```javascript
Opening Web Socket...
stomp.js?f746:134 Web Socket Opened...
stomp.js?f746:134 >>> CONNECT
login:mylogin
passcode:mypasscode
client-id:my-client-id
accept-version:1.1,1.0
heart-beat:10000,10000

<<< CONNECTED            // 服务器推送链接链接成功事件
version:1.1
heart-beat:0,0

connected to server undefined
stomp.js?f746:134 >>> SUBSCRIBE  // 客户端发送订阅请求
id:sub-0
destination:/topic/public_news

<<< MESSAGE            // 服务器推送消息
destination:/topic/public_news
content-type:text/plain;charset=UTF-8
subscription:sub-0
message-id:2voe0bmd-45
content-length:220

["清华大学16位学霸为一事现身PK 简历吓坏网友 - 738","闹市区树上长出6斤多蘑菇 保安大叔炒着吃了(图) - 415","女司机第2次上路撞死过路老人 教练被抓走 司法频道 - 491"]

```

## 总结

后端发送代码:
```java
template.convertAndSend("/topic/public_news", JSON.toJSONString(list));
```

前端订阅代码
```java
this.varStore.stomp.subscribe('/topic/public_news', message => {
    let news = JSON.parse(message.body)
    // 把获取到的列表赋值给该变量，页面中会循环出该信息
    this.publicNews = news
})
```

一定要注意这里的地址，广播地址，前后都一致。后面的点对点地址前后是不一样的。



    