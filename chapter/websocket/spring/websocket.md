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

403 错误，一般来说链接上了服务器，但是拒绝服务；这里的问题就是前后分离的方式开发的，两个项目的域名和端口不一样，导致跨域了。


## 配置允许链接的源
从Spring Framework 4.1.5开始，WebSocket和SockJS的默认行为是**仅接受相同的源请求**。也可以允许所有或指定的起始列表。此检查主要是为浏览器客户端设计的。

3种可能的行为是：

 * 只允许相同的原始请求（默认）：在此模式下，当启用SockJS时，将Iframe HTTP响应头X-Frame-Options设置为SAMEORIGIN，并禁用JSONP传输，因为它不允许检查请求的来源。因此，当启用此模式时，不支持IE6和IE7。

* 允许指定的起始列表：每个提供的允许的起始必须以`http://` 或`https://`开始。在这种模式下，当启用SockJS时，基于IFrame和JSONP的传输都被禁用。因此，启用此模式时，不支持IE6至IE9。

* 允许所有来源：启用此模式，您应该提供*作为允许的原始值。在这种模式下，所有的运输都可用。

所以这里我们在后端配置中配置允许的源
**cn.mrcode.javawebsocketdemo.websocket.ws.WebSocketConfig**

```java
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        // 添加处理器
        registry.addHandler(myHandler(), "/myHandler")
                .setAllowedOrigins("*")
        ;
    }
```
`.setAllowedOrigins("*")`:这里配置成允许所有的host访问。当然你也可以配置只允许`http://mydomain.com`的请求链接

配置完成后，重启后端服务重新刷新页面，就能看到已经链接成功了。

后台则会打印日志
```java
cn.mrcode.javawebsocketdemo.websocket.ws.MyHandler handleTextMessage 19    - 收到消息：sessionId=0,msg=TextMessage payload=[发送数据], byteCount=12, last=true]
```
这个是由于在前端onopen链接成功后还发送了一个字符串到后端`ws.send('发送数据')`. 

那么问题来了，后端怎么发送消息给前端呢？


## 后端发送消息

后端在接收到消息之后，还想给前端发送一个交互消息。这里就很简单了。

`cn.mrcode.javawebsocketdemo.websocket.ws.MyHandler` 中我们覆盖了一个接收消息的方法。
```java
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        log.info("收到消息：sessionId={},msg={}", session.getId(), message);
    }
```

`WebSocketSession session` : 注意这个对象，顾名思义是一个session对象，这个session表示的意思是，一个链接对应一个session。前端中 `new WebSocket` 则是一个不同的session。 

所以这里的session和mvc中的session不是同一个概念。那么信息的发送也是通过这个session。
```java
 session.sendMessage(new TextMessage("Hello Word"));
```
