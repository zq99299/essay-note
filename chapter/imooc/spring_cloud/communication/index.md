# 应用间通信-框架与基本使用
比较HTTP REST 和 REST，同步和异步, 介绍Spirng Cloud 采用的两种HTTP方式，重点介绍Feign. 实例演示下单流程. 引出异步通信的思考.

由于视频内容太短，写在一个笔记文件中
本页包含内容：

* HTTP vs RPC
* RestTemplate的三种使用方式
* 负载均衡器：Ribbon
* 追踪源码自定义负载均衡策略
* Feign的使用

## HTTP vs RPC
* Dubbo
* SpringCloud ： 微服务之间使用http方式，可以夸语言

## RestTemplate的三种使用方式

```java
package cn.mrcode.imooc.spring.cloud.order.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.client.loadbalancer.LoadBalancerClient;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

/**
 * restTemplate 的三种用法
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/16 22:12
 */
@RequestMapping("/test")
@RestController
public class DemoClientController {
    @GetMapping("/word1")
    public String word1() {
        // 1. 直接使用restTemplate，url写死
        RestTemplate restTemplate = new RestTemplate();
        String result = restTemplate.getForObject("http://localhost:8080/test/hello", String.class);
        return result;
    }

    //~~==============================  方法2 ==============
    @Autowired
    private LoadBalancerClient loadBalancerClient;

    @GetMapping("/word2")
    public String word2() {
        // 2. 使用 LoadBalancerClient 通过应用名获取url，然后再使用restTemplate
        // 可以不用写ip地址了，解决了有多个服务的问题，但是还是有一点麻烦
        ServiceInstance si = loadBalancerClient.choose("product");
        String url = String.format("http://%s:%s", si.getHost(), si.getPort());
        RestTemplate restTemplate = new RestTemplate();
        String result = restTemplate.getForObject(url + "/test/hello", String.class);
        return result;
    }

    //~~==============================  方法3 ==============
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/word3")
    public String word3() {
        // 3. 使用@LoadBalanced注解 + RestTemplate 简化第2种里面的步骤

        // 这里传递的是服务名，不需要使用ip和端口号了
        String result = restTemplate.getForObject("http://product/test/hello", String.class);
        return result;
    }
}

```

## 客户端负载均衡器：Ribbon

向服务器拉取所有注册的服务，然后命中服务；使用到ribbon的库有：

* RestTemplate
* Feign
* Zuul

ribbon实现软负载均衡的核心有以下三点：

1. 服务发现
2. 服务选择规则 ：根据规则策略从众多服务中选择一个
3. 服务监听 ：剔除无效服务等

主要组件有：

* ServerList ：获取所有可用列表
* IRule ：通过规则选择一个服务
* ServerListFilter ： 过滤部分实例

默认是轮训的策略，如果有需要修改的话，可用查看官网文档，进行配置
> 自定义负载均衡的策略
> http://cloud.spring.io/spring-cloud-static/Finchley.SR1/single/spring-cloud.html#_customizing_the_ribbon_client_by_setting_properties

这里贴一下yml的配置
```yml
users:
  ribbon:
    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.WeightedResponseTimeRule
```


## Feign的使用

需要线增加依赖
```
compile('org.springframework.cloud:spring-cloud-starter-openfeign')
```

开启 EnableFeignClients
```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class OrderApplication {
```

编写接口映射
```java
package cn.mrcode.imooc.spring.cloud.order.controller;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

@FeignClient(name = "product")   // 服务名
public interface FeignClientServer {
    @GetMapping("/test/hello")
        // 定义需要调用的接口
    String hello();
}

```

使用
```java
@Autowired
private FeignClientServer feignClientServer;

@GetMapping("/word4")
public String word4() {
    return feignClientServer.hello();
}
```

### feign总结

* 声明式REST客户端（伪rpc）
* 采用了基于接口的注解
