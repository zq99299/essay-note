# SpringSecurityOAuth核心源码解析

![](/assets/image/imooc/spring_secunity/snipaste_20180807_134611.png)

要自己实现，必须查看源码，才能知道在哪里加东西

* TokenEndpoint ：整个流程入口点
* ClentDetailsService ： 读取第三方应用信息
* TokenRequest ： 封装了提交的其他参数信息
* TokenGranter ： 令牌授权者，找到一个授权模式进行处理
  都会产出两个对象
  - OAuth2Request ：处理之后的新对象
  - Authentication ： 谁在授权的信息，对应用户信息
* OAuth2Authentication : 哪一个用户在对哪一个应用进行授权
* AuthorizationServerTokenServices：
  - TokenStore ： 令牌存储
  - TokenEnhancer ：令牌加强器，可以对令牌进行改造加强
