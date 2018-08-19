# 服务容错 下

* feign-hystrix
* hystrix-dashboard
* zuul 超时配置

## feign-hystrix

要在配置中开启该功能
```yml
feign:
  hystrix:
    enabled: true
```
这里没有在订单服务中去尝试了，直接写步骤，以前测试过这个流程

在之前的FeignClient上，增加一个 fallback 参数，需要配置一个class；

这个class其实就是 SchedualServiceHi 的实现类，当遇到服务降级的时候，则会调用指定类型的实例的同名方法
```java
@FeignClient(value = "service-hi", fallback = SchedualServiceHiHystric.class)
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi", method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}

@Component
public class SchedualServiceHiHystric implements SchedualServiceHi {
    @Override
    public String sayHiFromClientOne(String name) {
        return "sorry " + name;
    }
}
```

## hystrix-dashboard
> http://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.0.1.RELEASE/single/spring-cloud-netflix.html#_hystrix_metrics_stream

> http://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.0.1.RELEASE/single/spring-cloud-netflix.html#_hystrix_timeouts_and_ribbon_clients


加入依赖：

```
compile 'org.springframework.cloud:spring-cloud-starter-netflix-hystrix-dashboard'
```

在启动类上开启HystrixDashboard
```java
@EnableHystrixDashboard
```
启动项目后，在控制台会看到`/hystrix` 映射

访问：http://localhost:9001/hystrix 会出现一个豪猪页面。
![](/assets/image/imooc/spring_cloud/snipaste_20180819_235309.png)

页面中的提示到，如果是单机可以使用： Single Hystrix App: http://hystrix-app:port/actuator/hystrix.stream

在这里我填入：http://localhost:9001/actuator/hystrix.stream

进入后，大大的红字显示无法连接：Unable to connect to Command Metric Stream.

查看上面列出来的文档，发现需要把 hystrix.stream 暴露出来；之前讲到过这个，可以设置为 * 把所有的都暴露出来
```yml
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```
配置了之后还是无法连接，也无法点击 include。 最后还是添加了以下依赖
```
compile 'org.springframework.boot:spring-boot-starter-actuator'
```
![](/assets/image/imooc/spring_cloud/snipaste_20180819_235255.png)

注意：这里面的报告显示的访问过的，如果没有访问则不会显示；
比如图中的test2.test3 前面演示的时候写的HystrixCommand。访问几次就会出来了


关于多次访问某一个服务，触发熔断，可以使用post的 runner功能来指定请求某一组连接多少次；
前提是，必须把一个请求先保存到一个collection中

## zuul 超时配置

在启动后第一次访问的时候，有可能会超时。

原因是：类有延迟初始化的情况，所以会造成这样的情况发生，这个很好理解。包括你写的静态工具类的静态代码块一样

查看zuul的依赖了hystrix。可以配置下默认的超时时间，设置大一点就好了

```yml
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 1000
```
