# stomp-新闻展示示例

大体上做以下的功能：

1. 所有用户都默认订阅 公共广播新闻，所有用户都能收到，可以做全局通知
2. 提供几个新闻类别，只有用户点击该类别的订阅，我们才会主动的推送该订阅消息
3. 提供用户取消订阅功能，取消订阅之后，则不能再继续推送了。
4. 用户离开页面后，自动取消推该用户的订阅消息推送

## 本节说明

由于是一个小例子，不会把每个步骤都贴上来。我只会记录下关键的配置代码和实现思路。

一般我是喜欢先写页面布局，接下来看看这个页面布局长什么样子

## 页面布局
![](/assets/image/websocket/spring/stomp页面布局.png)

如上图所示：要做的功能有以下几点：

1. 链接：点击链接服务按钮：链接上后端ws服务
2. 断开链接：链接上服务之后，链接服务按钮会被替换成 断开链接的按钮：切断链接后，后端则不再继续推送该数据到该用户的订阅地址上
3. 广播订阅：服务链接上之后，自动订阅公共新闻
3. 点对点订阅：动漫和八卦类目需要用户自己订阅：订阅哪一种则接收哪一种类型的数据
4. 点击其他页面，在控制台(F12)打印断开链接的消息。后端也要相应的不推送该用户的所有消息

接下来我们一步一步的做；

## 链接服务和断开

后端配置,和前面一章节的一样,这里我们从最简单的需求开始做起。然后不断完善改程序
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/stomp").setAllowedOrigins("*")
                .withSockJS()
                .setMessageCodec(new FastjsonSockJsMessageCodec())
        ;
    }
}
```

前端主要的js代码。

```javascript
<script>
  import SockJS from 'sockjs-client'
  import Stomp from 'stompjs'

  export default {
    data () {
      return {
        msg: {
          status: 0,
          info: '~'
        },
        connectLoading: false,
        isConnect: false
      }
    },
    created () {
      this.varStore = {
        stomp: null // 链接实例
      }
    },
    mounted () {

    },
    methods: {
      // 链接到后端服务
      connect () {
        this.connectLoading = true
        // 注意这里的链接地址
        let ws = new SockJS('http://localhost:8082/stomp')
        let stomp = Stomp.over(ws)

        // 注意这里的header 暂时不是必须的。
        let headers = {
          login: 'mylogin',
          passcode: 'mypasscode',
          // additional header
          'client-id': 'my-client-id'
        }
        stomp.connect(headers,
          () => {
            this.varStore.stomp = stomp
            this.connectLoading = false
            this.msg.status = 1
            this.msg.info = '链接成功'
            this.isConnect = true
          },
          (error) => {
            this.connectLoading = false
            this.msg.status = 0
            this.msg.info = `链接失败: ${error}，或许是后端服务未开启`
            this.isConnect = false
          })
      },
      // 端口链接
      disconnect () {
        this.varStore.stomp.disconnect()
        this.msg.status = 0
        this.msg.info = `链接断开: 手动断开`
        this.isConnect = false
      }
    }
  }
</script>
```

基础代码完成之后，接下来重点来了。怎么进行广播订阅。一对多

















