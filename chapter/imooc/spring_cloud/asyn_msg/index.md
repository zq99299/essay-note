# 异步和消息

异步：客户端请求不会阻塞进程，服务端的响应可以是非及时的

异步的场景形态：

* 通知
* 请求/异步响应
* 消息

MQ应用场景

* 异步处理 ：如用户积分，通过消息异步处理积分逻辑
* 流量削峰 ：控制参与活动的人数
* 日志处理 ：最典型的就是kafka
* 应用解耦 ：
  - 订单服务下单后，需要通知商品服务扣减库存：可以使用消息
  - 商品服务订阅这个消息，完成自身的库存扣减


## RabbitMQ的基本使用
把之前的order使用配置中心配置还原。我们学习的开发环境下使用配置中心有点麻烦；

引入mq消息；默认会应用rabbitmq包
```
compile('org.springframework.boot:spring-boot-starter-amqp')
```

编写消息接收
```java
package cn.mrcode.imooc.spring.cloud.order.message;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * mq接收
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/18 22:21
 */
@Slf4j
@Component
public class MqReceiver {
    @RabbitListener(queues = "myQueue")
    public void process(String message) {
        log.info("接收到消息 myQueue：{}", message);
    }
}

```

编写个测试发送的测试用例
```java
父类里面只是有两个注解而已：
@RunWith(SpringRunner.class)
@SpringBootTest
-------------------------------------------

public class MqReceiverTest extends OrderApplicationTests {
    @Autowired
    private AmqpTemplate amqpTemplate;

    @Test
    public void send() {
        amqpTemplate.convertAndSend("myQueue", new Date());
    }
}
```

启动项目会发现异常了：myQueue没有被定义；

```java
org.springframework.amqp.rabbit.listener.BlockingQueueConsumer$DeclarationException: Failed to declare queue(s):[myQueue]

```

## `@RabbitListener` 的其他用法

下面的示例当然只能用其中一个了。全部写出来，只是为了把 导入的注解所属全路径拉出来
```
package cn.mrcode.imooc.spring.cloud.order.message;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class MqReceiver {
    @RabbitListener(queues = "myQueue")
    // 当队列不存在的时候，自动创建队列
    @RabbitListener(queuesToDeclare = @Queue("myQueue"))
    // 自动创建队列和Exchange 并绑定
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("myQueue"), exchange = @Exchange("myExchange")
    ))
    public void process(String message) {
        log.info("接收到消息 myQueue：{}", message);
    }
}

```
