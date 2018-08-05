# SpringSocial简介

spring security 与 oath social的关系

还是上次那个图片示例，如果只是获取微信的昵称头像信息，

在spring security中被认定为认证成功的标志是：根据用户信息构建Authentication放入SecurityContext中

所以只要引导用户走完oath的所有流程。最后根据用户信息构建Authentication放入SecurityContext中

social的原理：
基于我们之前学习过的过滤链原理，在过滤器链上增加了一个 SocialAuthenticationFilter，

拦截到有需要第三方登录的请求则开始引导完成所有的流程，就完成了第三方登录

## social基本概念和原理
之前1-5步都是协议化流程步骤

这里只是介绍的是与我们要写代码相关的流程；实现这些节点就可以运行了。
![](/assets/image/imooc/spring_secunity/snipaste_20180806_001353.png)

* OAuth2Operations（OAuth2Template） ： 封装了1-5的步骤
* Api（AbstractOAuth2ApiBinding） 对第6步提供了支持
* Connection （OAuth2Connection）  包含用户信息的对象，
* ConnectionFactory（OAuth2ConnectionFactory）
  - ServiceProvider 创建Connection，要走1-5的流程，所以包含ServiceProvider
  - ApiAdapter  OAuth2Connection是固定结构的数据，对第三方api返回的数据进行匹配；读取用户信息

问题：
* 那么服务提供的信息是如何与业务系统中的用户是如何关联的呢？

  在 social 中是存在数据库中的，存放的是业务系统的userid月服务商用户的一个对应关系；

* 由谁来操作这个数据库中的表呢？

  UsersConnectionRepository（JdbcUsersConnectionRepository）

> 官网 ： https://projects.spring.io/spring-social/

官网中页面信息提供了

* Main Projects 官网已发布的项目，如连接Facebook的项目（上面讲的基本上都实现了，可能只需要简单的配置即可）
* Incubator Projects 孵化中的项目，也就是正在开发中的
* Community Projects 社区项目，非官网提供，但是放在这里应该质量还算是比较好的吧

该课程会接收QQ和微信登录

* qq的oath协议实现比较标准；使用scocial实现qq登录
* 微信的oath协议不是很标准，有特性化的在里面；重点关注怎么处理特性化的
