# SpringSecurityOAuth简介
![](/assets/image/imooc/spring_secunity/snipaste_20180807_100541.png)

传统方式：基于session
* 开发繁琐
  - 基于cookie：传统方式是容器和浏览器自动处理的cookie
* 安全性和客户体验差
* 有些前端技术不支持cookie，如小程序

基于token方式：oauth
* 参数中携带token
* 可以对token更大程度的控制

![](/assets/image/imooc/spring_secunity/snipaste_20180807_101309.png)




SpringSecurityOAuth封装了服务提供商大部分的操作；而social则是封装了客户端和服务提供商交互的流程

![](/assets/image/imooc/spring_secunity/snipaste_20180807_101535.png)

协议中没有规定token要怎么生成和存储。spring oath中规定了；
除了4种的标准模式；让我们自己的自定义验证也添加到该流程中，相当于自定义认证？

本章内容简介：

* 实现一个标准的OAth2协议中的Provider角色的主要功能
* 重构之前的三种认证方式的代码，使其支持token
* 高级特性
