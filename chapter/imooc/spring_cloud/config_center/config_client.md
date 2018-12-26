# config client 的使用

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [config client 的使用](#config-client-的使用)
	- [配置中心的高可用](#配置中心的高可用)
	- [当服务注册中心端口不是默认的8761时](#当服务注册中心端口不是默认的8761时)
	- [热更新测试](#热更新测试)

<!-- /TOC -->

按order项目来改造

添加依赖
```
compile('org.springframework.cloud:spring-cloud-config-client')
```

配置yml。注意是新建 **bootstrap.yml** ；是一个特殊的引导文件。能保证在项目启动最开始执行这里的配置，
然后开始启动的时候，项目中依赖环境配置的实例对象才能正确的加载到下载的配置文件

```yml
spring:
  application:
    name: order
  cloud:
    config:
      discovery:
        enabled: true  # 配置发现打开，默认是关闭的
        service-id: config-server # config-server 项目的 spring.application.name
      profile: dev  # 环境 spring.application.name + 这里的profile 组成一个之前服务端提供的获取地址参数
```

然后启动项目会发现控制台日志信息，去配置中心获取配置文件了
```
2018-08-18 10:33:08.672  INFO 18248 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at : http://localhost:9050/
2018-08-18 10:33:11.590  INFO 18248 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Located environment: name=order, profiles=[dev], label=null, version=4f6a63483faf38d1e2bd83e889edbda9637b3289, state=null
2018-08-18 10:33:11.590  INFO 18248 --- [           main] b.c.PropertySourceBootstrapConfiguration : Located property source: CompositePropertySource {name='configService', propertySources=[MapPropertySource {name='configClient'}, MapPropertySource {name='https://gitee.com/zhuqiang/imooc-spring-cloud-config-repo.git/order-dev.yml'}, MapPropertySource {name='https://gitee.com/zhuqiang/imooc-spring-cloud-config-repo.git/order.yml'}]}
2018-08-18 10:33:11.596  INFO 18248 --- [           main] c.m.i.s.cloud.order.OrderApplication     : No active profile set, falling back to default profiles: default
```

还有一个需要注意的是：获取配置中心的配置的时候;如 `http://localhost:9050/order-dev.yml`
会发现配置中心的控制台中打印了两个文件的信息；如果你两个文件内容不一致的话，会发现他们的内容被合并了

这里你要明白：我们之前在本地使用配置文件的方式是什么，这里获取也是一样的道理，不带后缀的是公共的配置，带后缀的是环境配置
```
2018-08-18 11:06:08.608  INFO 1704 --- [nio-9050-exec-3] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: file:/C:/Users/mrcode/AppData/Local/Temp/config-repo-1121197125805238075/order-dev.yml
2018-08-18 11:06:08.608  INFO 1704 --- [nio-9050-exec-3] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: file:/C:/Users/mrcode/AppData/Local/Temp/config-repo-1121197125805238075/order.yml
```

## 配置中心的高可用
因为配置中心本身就是一个微服务，所以只需要多启动几个服务即可

## 当服务注册中心端口不是默认的8761时
当服务注册中心端口不是默认的8761时，上面 bootstrap.yml 中的配置就不能满足需求了；

因为客户端在启动的时候需要去服务注册中心获取配置中心的信息；

这个时候的解决访问就是；把eureka的配置放在  bootstrap.yml 中

**注意：** bootstrap.yml 也可以分环境，但是必须 公共 bootstrap.yml 文件必须存在。才能添加 bootstrap-dev.yml;
这样一来对于 eureka 的配置也能分环境加载了

**注意：** 配置文件可以混合部署； 如：可以只把 prod 配置放在配置中心，dev 的就放在项目中

## 热更新测试

当初使用配置中心的理由之一就是：需要不启动机器，配置也能进行热更新

那么来测试下：

1. 在order中提供一个测试服务，获取配置文件中的某一个值
2. 更改服务器配置后，再次访问，对比是否能获取到最新的值

```java
@RestController
public class EnvController {
    @Value("${env}")
    private String env;

    @GetMapping("/env/print")
    public String env() {
        return env;
    }
}
```
访问：http://localhost:9001/env/print 查看打印信息

这里测试结果肯定是不可以的。

下一章节讲解如何使用 springCloud Bus自动刷新配置
