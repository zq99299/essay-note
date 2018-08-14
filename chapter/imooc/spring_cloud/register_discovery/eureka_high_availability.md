# Eureka的高可用

当服务注册中心挂掉的时候，那么就存在单点故障了；

这节怎么搭建服务注册中心集群模式；

原理：两个服务注册中心，相互注册

另外再增加一个项目，方便操作。在同一个项目里面修改端口也是可以的；
一个里程碑版本一个正式版本的spring cloud 配置相差很大？

两个项目除了端口号不同，其他的都相同
```yml
eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: true   # 把自己当做实例注册，这个一定要开启
    service-url:
      defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/
  server:
    enable-self-preservation: false
spring:
  application:
    name: eureka
server:
  port: 8761
```

客户端只在8761上注册，但是会在8762上也能看到，这就集群成功了；

* register-with-eureka: 要开启把自己的实例注册到指定的服务地址上
* defaultZone ：

  对于单个服务中心来说，register-with-eureka可以关闭，因为不需要注册在自己上面，
  对于多个服务中心来说，把自己注册到其他服务注册中心即可


针对于客户端来说，在某一台上注册，其他服务注册中心也能共享到这个客户端，
但是当指定的服务注册中心挂掉之后，这个客户端再启动的话，则不能注册了。

所以客户端可以把所有服务中心都注册上。

## 有一点不太明白的问题

1. 我这里 UNKNOWN 的实例，端口是8080，只要服务注册中心启动过一会就会自动注册，不知道是什么鬼
2. 关于客户端关闭的时候，在服务注册中心不会踢掉，不知道是什么情况，百度说和租约什么的有关
3. 暂时不要管这些，等看完视频看看这个老师的教学程度怎么样。
