# SpringCloudStream
> https://cloud.spring.io/spring-cloud-stream/
>
> https://docs.spring.io/spring-cloud-stream/docs/Elmhurst.SR1/reference/htmlsingle/#_spring_cloud_stream_core

![](/assets/image/imooc/spring_cloud/SCSt-overview.png)

* 目标绑定器：负责提供与外部消息传递系统集成的组件。
* 目标绑定：外部消息传递系统和应用程序之间的桥接消息的生产者和消费者（由目标绑定器创建）。
* 消息：生产者和使用者用于与目标绑定器（以及通过外部消息传递系统的其他应用程序）通信的规范数据结构。

对消息中间件的进一步封装，应用层无感知。甚至可以无感知切换消息中间件；

目前只支持 : RabbitMQ and Apache Kafka

加入依赖：
```
compile 'org.springframework.cloud:spring-cloud-starter-stream-rabbit'
```

定义接口：

注意这里的接口定义：output，不能和input定义成相同的名称，启动会报错
官网文档中的示例也是不一样的；
```java
package cn.mrcode.imooc.spring.cloud.order.message;

import org.springframework.cloud.stream.annotation.Input;
import org.springframework.messaging.SubscribableChannel;

/**
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/18 23:39
 */
public interface StreamClient {
    String INPUT = "myMessage";

    @Input(INPUT)
    SubscribableChannel input();

    // 注意：在同一个应用中，不能存在两个myMessage的实例
    // 测试的时候是不能这样写的。启动就会报错，已经存在了一个myMessage实例
    // 发送的最后写在了另外一个程序中
//    @Output(INPUT)
//    MessageChannel output();
}


```

定义接收端
```java
package cn.mrcode.imooc.spring.cloud.order.message;

import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.stereotype.Component;

/**
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/18 23:42
 */
@Component
@EnableBinding(StreamClient.class) // 开启绑定接口
@Slf4j
public class StreamReceiver {
    // 绑定输入
    @StreamListener(StreamClient.INPUT)
    public void process(String message) {
        log.info("StreamListener process message {}" ，message);
    }
}

```

测试发送：
测试也是使用的input。这里的input和output应该只是语义，因为发送方往往是另外一个系统；

系统中专门定义output，用来发送， input用来接收

```java
package cn.mrcode.imooc.spring.cloud.order.message;

import cn.mrcode.imooc.spring.cloud.order.OrderApplicationTests;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.support.MessageBuilder;

import java.util.Date;


public class StreamReceiverTest extends OrderApplicationTests {
    @Autowired
    private StreamClient streamClient;

    @Test
    public void send() {
        String msg = "now:" + new Date();
        // 这样发送，只能是 当前的这个应用能接收到消息
        // 并且不会经过消息队列
        streamClient.input().send(MessageBuilder.withPayload(msg).build());
    }
}
```

把发送的写在了商品服务里面

```java
package cn.mrcode.imooc.spring.cloud.product.message;

import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;

public interface StreamClientOutput {
    String OUTPUT = "myMessage";
    @Output(OUTPUT)
    MessageChannel output();
}

```
```java
package cn.mrcode.imooc.spring.cloud.product.controller;

import cn.mrcode.imooc.spring.cloud.product.message.StreamClientOutput;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;

@RestController
@EnableBinding(StreamClientOutput.class)
public class StreamController {
    @Autowired
    private StreamClientOutput streamClientOutput;

    @GetMapping("/send")
    public void send() {
        String msg = "now:" + new Date();
        streamClientOutput.output().send(MessageBuilder.withPayload(msg).build());
    }
}

```

再次测试，和观察rabbitmq管理界面，就会发现消息经过了消息队列，并被order中的input接收到了；

## 多order实例接收测试
开启两个order服务实例，开启方式可以如下图修改成不同的端口启动
![](/assets/image/imooc/spring_cloud/snipaste_20180819_104848.png)


你会发现，发送一条消息，会被两个队列消费，查看rabbitmq管理界面可看出原因如下：

1. 生成了2个临时队列
2. 都绑定在了名称为 myMessage 的 Exchange 上
3. 并且他们的 Routing key 都是 #

解决办法：只需要让两个实例订阅一个队列即可

## 消息应用分组（绑定一个队列）
```yaml
spring:
  application:
    name: order
  cloud:
    stream:
      bindings:
        myMessage:  # 对应的Exchange，该交换器绑定在order队列上
          group: order
```
再次启动两个order实例,会发现队列名称固定了，并且有两个消费者连接

![](/assets/image/imooc/spring_cloud/snipaste_20180819_105626.png)

## 复杂对象接收/发送

在接口中定义接收
```java
@Input(PERSON_INPUT)
SubscribableChannel personInput();
```

要发送或接收的消息对象
```java
package cn.mrcode.imooc.spring.cloud.product.message;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {
    private String name;
    private int age;
}
```
消息接收监听
```java
@StreamListener(StreamClient.PERSON_INPUT)
public void process(Person person) {
    log.info("StreamListener process message person {}", person);
}
```

发送方定义
```java
@Output(PERSON_OUTPUT)
MessageChannel personOutput();
```

发送
```java
@GetMapping("/send/person")
public void sendPerson() {
    Person person = new Person("mrcode", 18);
    streamClientOutput.personOutput().send(MessageBuilder.withPayload(person).build());
}
```

这样发送到消息队列中了。因为这里是自定义对象，那么在rabbitmq中是什么样的格式存储的呢？

查看办法：

1. 只启动发送方
2. 关闭接收方
3. 发送消息，然后在rabbitmq中的管理界面查看；

这里有一个注意的知识：

在yml中不配置group的时候，是每个实例都会创建一个临时的队列，
只有配置了group的实例运行过一次，队列才会一直存在。发送消息后，才能在队列中查看

查看结果是：放入队列中的是json字符串；

为什么是这样呢？

在配置group的时候，所属的对象是下面这个。里面有一个contentType属性，默认就是 MimeTypeUtils.APPLICATION_JSON

```java
org.springframework.cloud.stream.config.BindingProperties
```

## 消费完后回复一个消息

SendTo 注解，可以把返回的类容发送到绑定的端点上去
```java
  @StreamListener(StreamClient.PERSON_INPUT)
  @SendTo(StreamClient.INPUT)
  public String processAndReply(Person person) {
      log.info("StreamListener process message person {}", person);
      return "ok";
  }
```
