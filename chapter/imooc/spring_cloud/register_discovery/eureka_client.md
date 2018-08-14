# Eureka Client的使用
新建一个client项目；使用 ui 创建，选择 Cloud Discovery -> Eureka Discovery 即可

默认就是引用的 spring-cloud-starter-netflix-eureka-client 包，
而服务端是引用的 spring-cloud-starter-netflix-eureka-server，

这里要记住，ui会对应上具体的jar包。后面就就只贴出ui选择和依赖项

```
buildscript {
	ext {
		springBootVersion = '2.0.4.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'cn.mrcode.imooc.spring.cloud'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}


ext {
	springCloudVersion = 'Finchley.SR1'
}

dependencies {
	compile('org.springframework.cloud:spring-cloud-starter-netflix-eureka-client')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
	}
}

```

```java
@SpringBootApplication
@EnableEurekaClient  // 标识自己是一个客户端
public class ClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ClientApplication.class, args);
    }
}
```
配置和服务端类似
```yml
eureka:
  instance:
    hostname: localhost # 使用域名代替ip 效果 http://localhost:8762，可以是任意域名，但是本机不配置映射是不能访问的
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8762
spring:
  application:
    name: client
```

但是启动会报错，
```java
2018-08-14 22:45:13.800  WARN 228 --- [       Thread-5] .s.c.a.CommonAnnotationBeanPostProcessor : Invocation of destroy method failed on bean with name 'scopedTarget.eurekaClient': org.springframework.beans.factory.BeanCreationNotAllowedException: Error creating bean with name 'eurekaInstanceConfigBean': Singleton bean creation not allowed while singletons of this factory are in destruction (Do not request a bean from a BeanFactory in a destroy method implementation!)
```

最后也没有明白是什么问题，但是增加了web依赖就可以启动了

```
compile('org.springframework.boot:spring-boot-starter-web')
```

发现一个问题： 同一个client启动两次的话。会在服务注册中心出现一行红字;

原因：
* 他们会采用心跳机制进行检测client是否在线
* server在一定的时间内会统计client的上线率，低于一个阀值，则会出现警告

  有时候出现很快，有时候需要等待1分钟左右才会出现
  当出现这种情况的时候，会显示在线，而不是直接下线。

```
EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.
```

在开发的时候就会出现这样的问题。你以为在线，却有可能不在线了

关闭该保护策略：在server端
```
eureka:
  server:
    enable-self-preservation: false  # 关闭自我保护
```

为啥在实际测试总，也没有看到有下线呢？很奇怪
