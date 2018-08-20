# 服务容错 上
探讨熔断机制，Spring Cloud Hystrix的使用, Feign+Hystrix服务降级.

* 服务容错和 hystrix
* hystrix
* 服务降级
* 服务熔断
* 使用配置项


## 服务容错 和Hystrix

什么是雪崩效应？

简单来说：

```
a -> b -> c
```
当c出现问题的时候，b会同步等待并重试，时间异常，可能造成b也瘫痪，最终整个系统被拖垮




## Hystrix

spring cloud中的Hystrix，是防雪崩利器，基于netflix对应的hystrix; Hystrix 译意：豪猪

特点：

* 服务降级
* 依赖隔离
* 服务熔断
* 监控（Hystrix Dashboard）

## 服务降级 fallbackMethod
你或许见到过，比如在双11的时候，有时候会提示，网络不可用，开小差了等。

* 优先核心服务，非核心服务不可用或弱可用

  当流量过大的时候，可以保证支付，订单等核心服务可用，广告什么的可以先不可用

hystrix的使用步骤：

1. 通过HystrixCommand注解指定
2. fallbackMethod 中具体实现降级逻辑


测试方法：

先用RestTemplate请求接口
```java
@RestController
@RequestMapping("/hystrix")
public class HystrixController {
    private RestTemplate restTemplate = new RestTemplate();

    @GetMapping("/product/list")
    public String test1() {
        String result = restTemplate.getForObject("http://127.0.0.1:8080/product/list", String.class);
        // 其实这里只要抛出异常，也会触发断路器，服务降级
        return result;
    }
}
```
调用成功后，关闭商品服务；再次访问报错。
```
There was an unexpected error (type=Internal Server Error, status=500).
I/O error on GET request for "http://127.0.0.1:8080/product/list": Connection refused: connect; nested exception is java.net.ConnectException: Connection refused: connect
```

使用hystrix来完成服务降级的功能

加入依赖
```
compile 'org.springframework.cloud:spring-cloud-starter-netflix-hystrix'
```

开启断路器
```java
package cn.mrcode.imooc.spring.cloud.order;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.SpringCloudApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

// @SpringBootApplication
// @EnableEurekaClient
@EnableFeignClients
// @EnableCircuitBreaker  // 断路器
@SpringCloudApplication // 自带 @SpringBootApplication  @EnableDiscoveryClient @EnableCircuitBreaker
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}

```

```java
package cn.mrcode.imooc.spring.cloud.order.controller;

import com.netflix.hystrix.contrib.javanica.annotation.DefaultProperties;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

/**
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/19 21:14
 */
@RestController
@RequestMapping("/hystrix")
// 当有多个需要降级的时候，可以使用一个默认的降级服务
@DefaultProperties(defaultFallback = "defaultFallback")
public class HystrixController {
    private RestTemplate restTemplate = new RestTemplate();

    @HystrixCommand(fallbackMethod = "fallback")
    @GetMapping("/product/list")
    public String test1() {
        String result = restTemplate.getForObject("http://127.0.0.1:8080/product/list", String.class);
        return result;
    }

    private String fallback() {
        return "太拥挤了，请稍后再试";
    }

    private String defaultFallback() {
        return "默认提示：太拥挤了，请稍后再试";
    }
}


```

(商品服务还是停用状态哦)再次访问：http://localhost:9001/hystrix/product/list

## 超时设置

```java
@HystrixCommand(
        commandProperties = {
                @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")}
        ,
        fallbackMethod = "fallback"
)
@GetMapping("/product/list")
public String test1() throws InterruptedException {
    // 模拟用时2秒，可以访问下，你会发现只会访问一秒多后，就返降级返回了
    TimeUnit.SECONDS.sleep(2);
    String result = restTemplate.getForObject("http://127.0.0.1:8080/product/list", String.class);
    return result;
}
```
在不配置超时设置的时候，会发现睡眠2秒就会被降级，这是因为默认超时时间是1000毫秒

```java
从 HystrixCommand 注解点到源码中，然后找到了HystrixPropertiesManager
com.netflix.hystrix.contrib.javanica.conf.HystrixPropertiesManager#EXECUTION_ISOLATION_THREAD_TIMEOUT_IN_MILLISECONDS

看谁调用了这个名称，最后找到了 HystrixCommandProperties。里面默认设置的是1000毫秒
com.netflix.hystrix.HystrixCommandProperties

```

这里的休眠操作，应该放到商品服务中去模拟耗时，写在这里也发现一个有趣的事情：

当超时为1秒的时候，这里休眠两秒，浏览器会在1.5秒左右就返回了。而且休眠后面的代码不会继续执行，相当于被强制中断了一样

## 依赖隔离

* 线程池隔离 ：为每个HystrixCommand创建一个独立的线程池。

  当某一个HystrixCommand花生问题时，并不会影响其他的HystrixCommand

  自动实现了依赖隔离

## 服务熔断
服务熔断也是HystrixCommand命令配置的

效果是：在某一个时间段服务被降级次数达到N%，则在一段时间内直接降级服务，而不会去调用真实的服务

```java
@HystrixCommand(
        commandProperties = {
              // 这里大部分都是默认配置，来查看熔断器的效果
                @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
                // 必须在10秒内发生6个请求，才能使统计数据发挥作用； 默认参数是 20
                @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "6"),
                // 在5秒内直接服务降级
                @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "5000"),
                // 如果在10秒内超过50%的请求是失败的或潜在的，那么我们将跳闸电路
                @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50")
                // 上面的配置意思就是：在10秒内大于等于6个请求，当有超过3个失败的时候，则在5秒内后面访问该服务则直接服务降级返回
        }
)
@GetMapping("/product/list2")
public String test2(int num) throws InterruptedException {
    // 当传递2的时候直接成功，否则就休眠2秒触发服务降级，模拟服务调用失败效果
    if (num == 2) {
        return "success";
    }
    TimeUnit.SECONDS.sleep(2);
    String result = restTemplate.getForObject("http://127.0.0.1:8080/product/list", String.class);
    return result;
}
```

在微服务中，服务容错是很有必要的；

* 重试：在短时间内，服务调用失败，重试即可成功
* 熔断：对于更长时间调用失败来说，更多的尝试是没有意义的

> 断路器文章：https://martinfowler.com/bliki/CircuitBreaker.html

![](/assets/image/imooc/spring_cloud/snipaste_20180819_224359.png)

描述了断路器模式断路器机。

## 使用配置项

前面把配置写在代码中的。可以写在配置文件中。配置是很多的。并不是这里介绍的这么几种。

想要深入了解。只有在一定需求下，再去官网查看相关资料或则源码；

这里以超时设置为例；

在需要使用断路器的方法上加注解
```java
@HystrixCommand
@GetMapping("/product/list3")
public String test3() throws InterruptedException {
    TimeUnit.SECONDS.sleep(2);
    String result = restTemplate.getForObject("http://127.0.0.1:8080/product/list", String.class);
    return result;
}
```

配置文件yml
```yaml
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 1000

```

这就好了。说说这个配置是这么来的，因为没有idea的提示；

目前还不知道在源码的什么地方调用的。 但是可以看下注解的说明

hystrix.command 是开始，default 开始就是配置了。这里设置的是全局默认值。

如果要为某一个方法指定超时时间呢？看下面的commandKey源码说明；

会默认为你的方法名；

```java
public @interface HystrixCommand {
    /**
     * Hystrix command key.    // 看这里
     * <p/> // 看这里
     * default => the name of annotated method. for example:   
     * <code>
     *     ...
     *     @HystrixCommand
     *     public User getUserById(...)
     *     ...
     *     the command name will be: 'getUserById'
     * </code>
     *
     * @return command key
     */
    String commandKey() default "";

```
这里的方法名为： test3; 在配置文件中可以这样写
```yml
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 1000
    test3:
     execution:
        isolation:
          thread:
            timeoutInMilliseconds: 3000
```
