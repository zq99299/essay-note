# 异步处理rest服务

* 使用Callable异步处理rest服务
* 使用DeferredResult异步处理rest服务
* 异步处理配置


异步处理就是主线程使用委托副线程去处理业务，然后主线程去接纳其他的请求。提高性能

## Callable

![](/assets/image/imooc/spring_secunity/snipaste_20180802_174924.png)
```java
@RestController
public class AsyncController {
    private Logger logger = LoggerFactory.getLogger(getClass());

    @RequestMapping("/order")
    public Callable<String> order() {
        logger.info("主线程开始");
        Callable<String> result = () -> {
            logger.info("副线程开始");
            TimeUnit.SECONDS.sleep(1);
            logger.info("副线程返回");
            return "success";
        };
        logger.info("主线程返回");
        return result;
    }
}
```
输出

```
2018-08-02 17:43:24.406  INFO 15644 --- [nio-8080-exec-2] c.e.demo.web.async.AsyncController       : 主线程开始
2018-08-02 17:43:24.407  INFO 15644 --- [nio-8080-exec-2] c.e.demo.web.async.AsyncController       : 主线程返回
2018-08-02 17:43:24.414  INFO 15644 --- [      MvcAsync1] c.e.demo.web.async.AsyncController       : 副线程开始
2018-08-02 17:43:25.414  INFO 15644 --- [      MvcAsync1] c.e.demo.web.async.AsyncController       : 副线程返回
```

很奇怪。这个是spring mvc提供的支持吗？如果是不在spring框架中使用。自己写个测试例子，是在同一个线程中执行的

## DeferredResult
在实际场景中可能会非常复杂，假如如下入这样；下单服务的一个流程

![](/assets/image/imooc/spring_secunity/snipaste_20180802_174959.png)

线程1和线程2是隔离的。

业务实现逻辑如下：

1. 请求下单
2. 发送消息到消息队列中
3. 另外一个应用（假如是订单系统）处理下单，并返回处理结果

编写下单请求api

```java
@RestController
public class AsyncController {
    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private MockQueue mockQueue;
    @Autowired
    private DeferredResultHolder deferredResultHolder;

    @RequestMapping("/order")
    public DeferredResult<String> order() {
        logger.info("主线程开始");
        // 发送消息到消息队列，请求订单生成
        String orderNumber = RandomStringUtils.randomNumeric(8);
        mockQueue.setPlaceOrder(orderNumber);
        DeferredResult<String> deferredResult = new DeferredResult<>();
        // holder 只是用来存储 DeferredResult
        // 方便监听器线程拿到 DeferredResult
        deferredResultHolder.getMap().put(orderNumber, deferredResult);
        logger.info("主线程返回");
        return deferredResult;
    }
}
```

编写模拟队列
```java
@Component
public class MockQueue {
    private Logger logger = LoggerFactory.getLogger(getClass());
    private String placeOrder; // 请求下单
    private String completeOrder;  // 下单完成

    public String getPlaceOrder() {
        return placeOrder;
    }

    public void setPlaceOrder(String placeOrder) {
        // 模拟另外一个线程去执行耗时操作
        new Thread(() -> {
            logger.info("接到下单请求");
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            this.completeOrder = "下单请求处理完毕";
            this.placeOrder = placeOrder;
            logger.info(completeOrder + " : " + placeOrder);
        }).start();
    }

    public String getCompleteOrder() {
        return completeOrder;
    }

    public void setCompleteOrder(String completeOrder) {
        this.completeOrder = completeOrder;
    }
}
```

编写一个装载 DeferredResult 的缓存类；用来存储已经产生的DeferredResult

```java
@Component
public class DeferredResultHolder {
    private Map<String, DeferredResult<String>> map = new HashMap<>();

    public Map<String, DeferredResult<String>> getMap() {
        return map;
    }

    public void setMap(Map<String, DeferredResult<String>> map) {
        this.map = map;
    }
}
```

启动一个线程专门来监听消息队列处理结果，并且调用  deferredResult 返回结果
```java
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.stereotype.Component;

import java.util.concurrent.TimeUnit;

// ContextRefreshedEvent 当应用程序上下文初始化或刷新时引发的事件。
@Component
public class QueueLinstener implements ApplicationListener<ContextRefreshedEvent> {
    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private MockQueue mockQueue;
    @Autowired
    private DeferredResultHolder deferredResultHolder;

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        // 在线程中执行否则会阻塞监听器回调的线程
        new Thread(() -> {
            while (true) {
                // 当有订单完成时
                if (StringUtils.isNotBlank(mockQueue.getCompleteOrder())) {
                    String orderNumber = mockQueue.getPlaceOrder();
                    logger.info("返回订单处理结果：" + orderNumber);
                    deferredResultHolder.getMap().get(orderNumber).setResult("place order success");
                    mockQueue.setCompleteOrder(null);
                    deferredResultHolder.getMap().remove(orderNumber);
                } else {
                    try {
                        TimeUnit.MILLISECONDS.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
    }
}
```

访问api输出的结果如下:

```
2018-08-02 21:53:45.919  INFO 16944 --- [nio-8080-exec-2] c.e.demo.web.async.AsyncController       : 主线程开始
2018-08-02 21:53:45.920  INFO 16944 --- [nio-8080-exec-2] c.e.demo.web.async.AsyncController       : 主线程返回
2018-08-02 21:53:45.921  INFO 16944 --- [       Thread-8] com.example.demo.web.async.MockQueue     : 接到下单请求
2018-08-02 21:53:46.922  INFO 16944 --- [       Thread-8] com.example.demo.web.async.MockQueue     : 下单请求处理完毕 : 34633453
2018-08-02 21:53:47.269  INFO 16944 --- [       Thread-2] c.example.demo.web.async.QueueLinstener  : 返回订单处理结果：34633453
```
可以看到请求线程已经结束了，由其他线程进行处理并返回结果的；
这个套路就如同 socket里面reactor模型，主线程只接收请求，其他线程去执行逻辑；
但是有一个问题就是，容器里面肯定有一个线程接收请求，然后启用子线程去分发处理；然后这里还有搞多线程去处理，这个是什么套路呢？对性能有提高吗？


## 总结下
DeferredResult 在效果上实现了一个让主线程可以立即返回，但是连接不断开，可以通过DeferredResult设置返回结果来让连接返回并断开的功能

## 异步功能配置

在WebConfig中重写异步支持配置；从上面的例子中也看到了，主线程返回了。所以同步拦截器可能就不那么好用了；可以在这里配置
```java
public class WebConfig implements WebMvcConfigurer {
    @Autowired
    private TimeInterceptor timeInterceptor;

    @Override
    public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
        // 异步支持
//        configurer.registerCallableInterceptors(); // callable 拦截器
//        configurer.registerDeferredResultInterceptors(); // deferredResult拦截器
//        configurer.setTaskExecutor(); // 应该是自定义线程池
//        configurer.setDefaultTimeout() // 超时设置
    }
```
