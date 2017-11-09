# sockJsDemo

> 是一个协议，是websocket的备胎，一些浏览器不支持websocket的话，该协议则使用原始的ajax轮询等手段来模仿支持websocket的行为。

大致的能看出来。主要是在浏览器端用来解决某些浏览器不支持websocket功能。使用基本的模拟手段来尽可能的模拟出websocket功能。

所以这里配置会比较简单点


sockjs-client：NPM库链接 https://www.npmjs.com/package/sockjs-client

使用起来很简单，如下：

## 前端页面
```html
<template>
  <div>
    <p>状态： {{status}} </p>
    <input type="button" value="发送测试消息" @click="send" :disabled="!sock"/>
    <p>消息： {{msg}} </p>
  </div>
</template>
```
```javascript
<script>
  /**
   *
   * <pre>
   *  Version         Date            Author          Description
   * ---------------------------------------------------------------------------------------
   *  1.0.0           2017/11/07     zhuqiang        -
   * </pre>
   * @author zhuqiang
   * @version 1.0.0 2017/11/7 15:32
   * @date 2017/11/7 15:32
   * @since 1.0.0
   */
 import SockJS from 'sockjs-client'

  export default {
    data () {
      return {
        status: '未链接',
        msg: '',
        sock: null
      }
    },
    mounted () {
      let sock = new SockJS('http://localhost:8080/myHandlerSockjs')
      this.sock = sock
      sock.onopen = () => {
        this.status = '已链接'
      }
      sock.onmessage = (e) => {
        this.msg = '已收到消息：' + e.data
      }
      sock.onclose = () => {
        this.status = '链接已关闭'
      }

//      sock.close()
    },
    methods: {
      send () {
        this.sock.send('test')
        console.log('发送消息')
      }
    }
  }
</script>

```

上面的页面，提供了一个链接状态的展示，还提供了一个发送消息的按钮。接收消息的展示

怎么链接后端都是从`sockjs-client`库的教程中复制的链接代码。


访问该页面却发现报错了：

```javascript
ET http://localhost:8080/myHandler/info?t=1510041413326 404 (Not Found)
AbstractXHRObject._start @ abstract-xhr.js?c769:132
(anonymous) @ abstract-xhr.js?c769:21
:8081/#/sockjs-client:1 XMLHttpRequest cannot load http://localhost:8080/myHandler/info?t=1510041413326. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:8081' is therefore not allowed access. The response had HTTP status code 404.
```

上面的错误一共是两个。第一个错误，告诉我们这个链接没有访问到；第二个说是跨域了。先来解决第一个

这里注意`new SockJS('http://localhost:8080/myHandler')` 地址写的http,而不是ws。那么在后端我们也要配置上 对sockjs的支持。

## 后端配置

```java
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        // 这个是支持原生websocket的节点
        registry.addHandler(myHandler(), "/myHandler").setAllowedOrigins("*");
        // 对sockjs的支持节点
        registry.addHandler(myHandler(), "/myHandlerSockjs").setAllowedOrigins("*")
                .withSockJS()

        ;
    }
```

这里注意下，为什么注册了两个节点，是为了演示。如果只注册一个，那么必然会有另外一个链接不上。因为使用的协议不一致。


重启服务后，测试：可以链接上；但是发送消息的时候却报错了

```java
 java.lang.IllegalStateException: A SockJsMessageCodec is required but not available: Add Jackson 2 to the classpath, or configure a custom SockJsMessageCodec.
```

好把这个很明显的说明了，需要一个消息解码器，要么添加 `Jackson`库到classpath，要么自定义解码器

这里我尝试了下自定义的,因为我引入的是fastjson。发现该库还专门对spring的sockjs提供了支持

```java
        registry.addHandler(myHandler(), "/myHandlerSockjs").setAllowedOrigins("*")
                .withSockJS().setMessageCodec(new FastjsonSockJsMessageCodec());
```

到此sockjs也引入配置好了。但是现在还是原生websocket。因为你没有发现编码有什么改变。
如果想在一个链接上传递不同种类的消息，需要自己写消息类型，然后路由到不同的消息处理中去。

那么接下来就使用 stomp 来演示怎么通过消息路径来路由到我们指定的处理中去。


## 其他错误

这些错误在写本教程的时候并没有出现，不知道是不是因为一开始就使用了 高版本的spring的原因。之前出现过以下的错误，先记录着，有遇到的可以参考下


打开谷歌浏览器的控制台，查看会有一堆的错误提示出现，最开始的是下面的错误：
```
EventSource's response has a MIME type ("application/json") that is not "text/event-stream". Aborting the connection.
sockjs.min.js:2 Uncaught Error: Incompatibile SockJS! Main site uses: "1.1.4", the iframe: "1.0.0".
```

主主要的错误应该就是说 服务器端使用的版本是1.0.0,而js中使用的是1.1.4; 解决这个问题其实在[官网文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-fallback-xhr-vs-iframe)中已经写到了

解决方案：在配置类中配置和前端js使用相同版本的js
```java
public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
    // 这里开启sockjs协议支持
    registry.addHandler(new MyHandler(), "/api/myHandler").setAllowedOrigins("*")
    .addInterceptors(new HttpSessionHandshakeInterceptor())
    .withSockJS().setClientLibraryUrl("//cdn.jsdelivr.net/sockjs/1/sockjs.min.js");
}
```
上面的cdn指向的地址就是 [sockjs官网](https://github.com/sockjs/sockjs-client) 起步实例中的cdn文件。

**服务器端的错误则是：**
```java
Caused by: java.lang.IllegalArgumentException: Async support must be enabled on a servlet and for all filters involved in async request processing. This is done in Java code using the Servlet API or by adding "<async-supported>true</async-supported>" to servlet and filter declarations in web.xml. Also you must use a Servlet 3.0+ container
```

看这里要开启异步支持。由于我这里使用的是web.xml 所以在web.xml中开启,

这个在[官网文档中提到过要开启异步支持](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-fallback-sockjs-servlet3-async)

在网上查了下 web.xml中开启方法如下;
```xml
    <filter>
        <filter-name>httpPutFormFilter</filter-name>
        <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
        <async-supported>true</async-supported>
    </filter>
     <servlet>
        <servlet-name>mvcDispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
        <async-supported>true</async-supported>
    </servlet>
```

只要有 `<filter>`的地方都要添加`<async-supported>true</async-supported>`,还有就是mvcDispatcher的入口处

### 失败：连接在收到握手响应之前关闭
最主要的是上面两个错误，解决完成之后就可以链接上并也能前端往后端发送信息了。在前端控制台中，又冒出来一个如下的错误：
```javascript
websocket.js?0f24:6 WebSocket connection to 'ws://localhost:9105/api/myHandler/761/czzeyw3z/websocket' failed: Connection closed before receiving a handshake response
```

出现这个错误后，但是能正常的通信；只是在打印open之前报错了。后来找到了问题；

我使用的是vue-cli开发，用了代理转发到后台，所以在链接的时候使用了前端的地址，让代理转发的。代码如下
```javascript
var sock = new SockJS('/api/myHandler')
```

把相对路径更改为后端服务的直接路径，成功链接上
```javascript
var sock = new SockJS('http://localhost:9104/api/myHandler')
```





