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





