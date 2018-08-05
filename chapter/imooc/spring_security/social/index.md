# 使用Spring Social开发第三方登录

从本章开始进入了oauth协议；这里的第三方登录也是oauth协议；

## OAuth协议简介

* OAuth要解决的问题
* OAuth协议中的各个角色
* OAuth运行流程

## OAuth要解决的问题

![](/assets/image/imooc/spring_secunity/snipaste_20180805_233924.png)

考虑这样一个场景：假如开发一个APP-慕课微信图片美化，需要用户存储在微信上的数据；

传统的做法是：拿到用户的用户名密码
问题：
* 应用可以访问用户在微信上的所有数据；（不能做到读授权一部分功能权限）
* 只能修改密码才能收回授权
* 密码泄露可能性提高

OAuth要使用Token令牌来解决以上的问题
* 令牌绑定权限
* 令牌有过期时间
* 不需要用户的原始密码

## OAuth协议中的各个角色
* 服务提供商 Provider
* 资源所有者 Resource Owner
* 第三方应用 Client
* 认证服务器 Authorization Server

  派发令牌
* 资源服务器 Resource Server

  验证令牌，并提供服务

## OAuth运行流程
![](/assets/image/imooc/spring_secunity/snipaste_20180805_233937.png)

同意授权是关键；只有同意了才会有接下来的流程

OAuth提供了4中授权方式

* 授权码模式 Authorization code
* 简化模式 implicit
* 密码模式 resource owner Password credentials
* 客户端模式 client credentials

### 授权码模式流程

![](/assets/image/imooc/spring_secunity/snipaste_20180805_234652.png)

该模式与其他三种模式不同，最重要的区别就是：同意授权这个动作，是在认证服务器上完成的，而其他的三种都是在第三方应用上完成的。

该模式是4中模式中最严格最完整的一种协议

### 简化模式
在授权后直接带回令牌
