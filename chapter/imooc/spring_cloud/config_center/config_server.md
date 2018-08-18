# 统一配置中心 - ConfigServer
本节内容
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [统一配置中心 - ConfigServer](#统一配置中心-configserver)
	- [统一配置中心概述](#统一配置中心概述)
	- [创建config-server项目](#创建config-server项目)
	- [config-server](#config-server)
	- [自定义git克隆在本地的路径](#自定义git克隆在本地的路径)

<!-- /TOC -->
## 统一配置中心概述
为什么需要统一配置中心？

* 不方便维护：多人开发
* 配置内容安全与权限 ： 针对线上配置来说，一般不会暴露给开发人员的，可以进行隔离
* 更新配置项目需要重启 ：可以热更新部分配置？


```
                                            ->     product
远端git          ->            config-server         
                                  ↑         ->     order
                                  ↓
                                本地git
```
* config-server 会从远端git获取配置文件，放在本地git中
  - 当远端git不能访问的时候，就可以使用本地git中的配置文件
* config-server 的配置会推送给 应用，然后引用实现热更新

## 创建config-server项目
boot ui选择：
```
Cloud Discover -> Eureka Discover
Cloud Config -> Config Server
```
依赖
```
dependencies {
	compile('org.springframework.cloud:spring-cloud-config-server')
	compile('org.springframework.cloud:spring-cloud-starter-netflix-eureka-client')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

application.yml
```
spring:
  application:
    name: config-server
eureka:
  instance:
    hostname: localhost
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 9050
```

注解配置；因为注册中心也属于一个微服务，都需要像注册中心注册
```java
@SpringBootApplication
@EnableEurekaClient
@EnableConfigServer
public class ConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}
}
```

## config-server
启动会报错:
```java
Caused by: java.lang.IllegalStateException: You need to configure a uri for the git repository
	at org.springframework.util.Assert.state(Assert.java:73) ~[spring-core-5.0.8.RELEASE.jar:5.0.8.RELEASE]
```

需要添加相关的一些git仓库的配置；

在码云（https://gitee.com/）上创建一个私有仓库：imooc-spring-cloud-config-repo

先把order项目的配置放在上面:
```
|- 先放在根目录下即可
  |- order.yml
```

然后配置远程仓库地址信息
```yml
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/zhuqiang/imooc-spring-cloud-config-repo.git
          username: 99299684@qq.com
          password: git密码
```

再次启动项目能看到多了很多mapping
```
// 默认使用master分支
* name ： 文件名
* profiles ：环境; 如 application-dev.yml 后面的dev就是环境，这个文件名格式也是环境文件

{[/{name}-{profiles}.properties],methods=[GET]}
{[/{name}/{profiles:.*[^-].*}],methods=[GET]}
{[/{name}-{profiles}.yml || /{name}-{profiles}.yaml],methods=[GET]}
{[/{name}-{profiles}.json],methods=[GET]}
{[/{name}/{profiles}/{label:.*}],methods=[GET]

// 指定分支
* label : git分支名

{[/{label}/{name}-{profiles}.yml || /{label}/{name}-{profiles}.yaml],methods=[GET]}
{[/{label}/{name}-{profiles}.properties],methods=[GET]}
{[/{label}/{name}-{profiles}.json],methods=[GET]}
{[/{name}/{profile}/{label}/**],methods=[GET]
{[/{name}/{profile}/{label}/**],methods=[GET]}
{[/{name}/{profile}/**],methods=[GET]

// 访问了下，提示没有公共变量
{[/key],methods=[GET]}
{[/key/{name}/{profiles}],methods=[GET]}

{[/encrypt],methods=[POST]}
{[/encrypt/{name}/{profiles}],methods=[POST]}
{[/decrypt/{name}/{profiles}],methods=[POST]}
{[/decrypt],methods=[POST]}
{[/encrypt/status],methods=[GET]}
```

访问刚才配置的order.yml文件；
```
// org.springframework.cloud.config.server.environment.EnvironmentController#yaml
// 访问到了这个控制器，[/{name}-{profiles}.yml || /{name}-{profiles}.yaml],methods=[GET]
http://localhost:9050/order-a.yml

// 还可以访问下面的，会返回json数据
http://localhost:9050/order-a.json

```
该控制器返回的配置对象，还有自动检测文件内容格式是否正确的功能；如果配置文件的格式不对的话，会报错


## 自定义git克隆在本地的路径
在项目启动打印日志里面你可能会看到这样的日志信息;那么该路径就是默认存储的
```
xxxx.NativeEnvironmentRepository  : Adding property source: file:/C:/Users/mrcode/AppData/Local/Temp/config-repo-1121197125805238075/order.yml
```

```yml
cloud:
   config:
     server:
       git:
         basedir: 路径，但是不要配置在现有项目目录下，因为会被清空文件的
```
