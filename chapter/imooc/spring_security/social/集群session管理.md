# 集群session管理

![](/assets/image/imooc/spring_secunity/snipaste_20180806_230844.png)

spring Security 是基于session的安全框架。所以就会有这个问题

```
// 该项目用来做浏览器端的所以需要有session
    // 提供集群环境下的session管理,也没有被管理到，需要自己添加
    //如果在启动的时候报错，可以通过配置 yml文件中 spring: session: store-type: none
    compile('org.springframework.session:spring-session:1.3.3.RELEASE')
```
org.springframework.boot.autoconfigure.session.StoreType 标识支持的类型，
上面设置成none其实就是使用原生j2ee单机服务器session这种模式

## 为上面使用redis？
1. 所有请求都要使用session
2. 有过期时间，Redis天生支持

依赖：特别注意:spring-session:1.3.3.RELEASE在高版本的spring boot autoconfig中已经不支持了；
需要分开引用下面的包

```
// 该项目用来做浏览器端的所以需要有session
// 提供集群环境下的session管理,也没有被管理到，需要自己添加
//如果在启动的时候报错，可以通过配置 yml文件中 spring: session: store-type: none
//    compile('org.springframework.session:spring-session:1.3.3.RELEASE')
compile('org.springframework.session:spring-session-core')
compile('org.springframework.session:spring-session-data-redis')
```

## 开始改造
依赖添加之后，更改配置文件

```
spring:
  session:
    store-type: redis
```
访问登录页面:/imocc-signIn.html; 发现验证码图片无法显示，报错了。没有序列化

```java
// 相关报错序列化问题的地方都记得修改
public class ImageCode extends ValidateCode implements Serializable{
    private static final long serialVersionUID = -703011095085705839L;
    private BufferedImage image;  // 看这里，是一个复杂的图片对象
```

怎么处理这个图片对象呢？考虑在存和取的地方下功夫

```java
cn.mrcode.imooc.springsecurity.securitycore.validate.code.impl.AbstractValidateCodeProcessor#save

private void save(ServletWebRequest request, C validateCode) {
    // 不保存图片对象到redis session中，无法序列化
    // 因为在验证的时候不需要图片对象
    ValidateCode code = new ValidateCode(validateCode.getCode(), validateCode.getExpireTime());
    sessionStrategy.setAttribute(request, getSessionKey(), code);
}
```

## 测试

1. 启动两个不同端口的项目
2. 在同一个浏览器中
3. 其中一个端口先登录 ：localhost:80
4. 然后新开一个页面直接打开另外一个端口访问 localhost:80/user/me

之前配置的所有功能正常，只是在session失效只允许一个登录这个功能，可能会有一点点的延迟
