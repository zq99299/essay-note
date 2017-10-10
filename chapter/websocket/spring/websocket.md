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

那么这个就是说只能用于websocket协议来访问。

如果说你想问这个链接为404，那么请检查spring-mvc的配置，和是否把WebSocketConfig所在的路径扫描了,如
```xml
<context:component-scan base-package="cn.mrcode.javawebsocketdemo.websocket"/>
```

官方说到 ：Spring的WebSocket支持不依赖于Spring MVC。WebSocketHandler 在[WebSocketHttpRequestHandler](https://docs.spring.io/spring-framework/docs/5.0.0.RELEASE/javadoc-api/org/springframework/web/socket/server/support/WebSocketHttpRequestHandler.html)的帮助下将其集成到其他HTTP服务环境中 相对比较简单。在这里我明白去研究其他的方式。

## 前端链接服务

前端使用的vue-cli开发，不懂的同学可能要费点事，但是不要慌。这里只记录重要的代码和思路。同样也会放在git上。

```javascript
<template>
  <div class="hello">
    状态：{{msg}}
  </div>
</template>

<script>
  export default {
    data () {
      return {
        msg: 'Welcome to Your Vue.js App'
      }
    },
    mounted () {
      let ws = new WebSocket('ws://localhost:8080/myHandler')
      ws.onopen = () => {
        this.msg = '链接成功.'
        // Web Socket 已连接上，使用 send() 方法发送数据
        ws.send('发送数据')
      }

      ws.onmessage = (evt) => {
        let receivedMsg = evt.data
        this.msg = '数据已接收...：' + receivedMsg
      }

      ws.onclose = () => {
        // 关闭 websocket
        this.msg = '连接已关闭...'
      }
    }
  }
</script>
```
上面的代码，在mounted函数中(看成一个钩子函数，进入该页面后，该函数会被执行)，里面的代码是标准的 [w3c-html5-websocket](https://www.w3cschool.cn/html5/html5-websocket.html) 内容。只不过被转成了es6的语法。 

这里代码展示的效果是，进入该页面的时候，会去链接到我们后端暴露的websocket服务，（要记得找一个高版本的浏览器，否则可能不支持websocket）。分别在他的链接成功/关闭/收到消息事件回调函数中，在页面显示不同的字符串。

好了问题来了，按照现在的代码，肯定是链接不上的。页面一闪而过然后显示`状态：连接已关闭...`; 这个时候你按F12打开浏览器控制台就会发现报错了。
```bash
WebSocket connection to 'ws://localhost:8080/myHandler' failed: Error during WebSocket handshake: Unexpected response code: 403
mounted @ HelloWorld.vue?8664:16
...
```

