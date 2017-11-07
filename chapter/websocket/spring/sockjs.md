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
    <input type="button" value="发送测试消息" @send="send" :disabled="!sock"/>
    <p>消息： {{status}} </p>
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
      let sock = new SockJS('http://localhost:8080/myHandler')
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
      }
    }
  }
</script>

```

上面的页面，提供了一个链接状态的展示，还提供了一个发送消息的按钮。接收消息的展示

怎么链接后端都是从`sockjs-client`库的教程中复制的链接代码。
