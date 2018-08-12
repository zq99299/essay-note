# Spring开发技巧
记录了该学习中遇到源码中不错的工具类。之前可能自己都没有见到过的；这里介绍的只是用到过的功能，没有用的功能不会写出来

* `org.springframework.social.connect.web.SessionStrategy`  session操作

  功能：session大部分操作都包括
* `org.springframework.util.AntPathMatcher` 路径匹配

  功能：给定2个url，判定是否匹配，支持srping中带通配符的url
* `org.springframework.beans.factory.InitializingBean`  在所有属性设置完成后，被调用一次该方法设置

## Bean条件注解
提供了多种不同的条件注解，可以查看该路径下的不同注解源码了解

** 使用场景: ** 使用方如果提供了自定义的实现则不使用这里的默认实现；只需要让spring容器中存在这里指定name的bean即可完成自定义覆盖默认配置的效果
  ```java
  路径：org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean

  @Bean
    // spring 容器中如果存在imageCodeGenerate的bean就不会再初始化该bean了
    // 条件注解，不指定name的话，应该是按类型来判定的
    // 所以spring中貌似是不指定name的
    @ConditionalOnMissingBean(name = "imageCodeGenerate")
    public ValidateCodeGenerate imageCodeGenerate() {
        ImageCodeGenerate imageCodeGenerate = new ImageCodeGenerate(securityProperties.getCode().getImage());
        return imageCodeGenerate;
    }
  ```

### `@ConditionalOnProperty`

根据配置创建bean；
```java
    @Bean
    @ConditionalOnProperty(prefix = "imooc.security.oauth2", name = "tokenStore", havingValue = "redis")
    public TokenStore tokenStore() {
        return new MyRedisTokenStore(redisConnectionFactory);
    }

    @Configuration
    // matchIfMissing ：当tokenStore没有值的时候是否生效
    // 当tokenStore = jwt的时候或则tokenStore没有配置的时候使用下面的配置
    @ConditionalOnProperty(prefix = "imooc.security.oauth2", name = "tokenStore", havingValue = "jwt", matchIfMissing = true)
    public static class JwtTokenConfig {
```
## Bean注解
```java
@Bean
@ConditionalOnMissingBean(name = "smsValidateCodeGenerator")
// 注意方法名称：如果没有指定bean则按方法名称作为beanName返回
public ValidateCodeGenerator smsCodeGenerate() {

以上代码默认的BeanName是：smsCodeGenerate
```

## BeanPostProcessor 初始化钩子方法
```java
org.springframework.beans.factory.config.BeanPostProcessor

@Component
public class SpringSocialConfigurerPostProcessor implements BeanPostProcessor {
    // 任何bean初始化回调之前
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    //任何bean初始化回调之后
    // 在这里把之前浏览器中配置的注册地址更改为app中的处理控制器
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        /**
         * @see SpringSocialConfig#imoocSocialSecurityConfig()
         */
        if (beanName.equals("imoocSocialSecurityConfig")) {
            SpringSocialConfigurer config = (SpringSocialConfigurer) bean;
            config.signupUrl("/social/signUp");
            return bean;
        }
        return bean;
    }
}

````

## 依赖查找
** 依赖查找：** 当有多个子类实现的时候，根据beanNam进行查找需要的子类；
如下图的autowired,声明是一个Map,那么key = beanName、value = 不同的子类实现

** 使用场景: ** 编写基础架子，提供了多种相同功能的不同实现策略的时候
```java
@Autowired
private Map<String, ValidateCodeGenerate> validateCodeGenerates;
```



## SpringBoot各种默认路径技巧
有时候你引入一些包，就会发现会增加多个restfull服务，这些服务注意查看控制台日志，就能找到对应的处理器

对于springboot默认提供处理错误的restfull服务如下
```java
// org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController
                      // BasicErrorController 类提供的默认错误信息处理服务
```

## Configuration条件注解

### ConditionalOnProperty
在`imooc.security.social.qq`前缀下，当appId属性存在的时候才让该配置生效
```java
@Configuration
// 当配置了app-id的时候才启用
@ConditionalOnProperty(prefix = "imooc.security.social.qq", name = "app-id")
public class QQAutoConfig extends SocialConfigurerAdapter {
```


## ``@FrameworkEndpoint``
该注解只用于框架提供端点，语义和 Controller 注解一致，
如果使用Controller提供了和FrameworkEndpoint标识的相同路径，则框架提供的端点不会被路由到；

一般的场景用于：覆盖框架默认提供的端点；


## `@Order` 注解

### 可以为依赖查找集合按顺序注入

比如以下场景，对于bean的注入顺序，可以按照order指定的来放入
```java
@Component
public class DefaultAuthorizeConfigManager implements AuthorizeConfigManager {
    // 由于需要有序的，所以不能再使用set了
    // 依赖查找技巧
    @Autowired
    private List<AuthorizeConfigProvider> providers;

@Component
@Order(Integer.MIN_VALUE)
public class CommonAuthorizeConfigProvider implements AuthorizeConfigProvider {

@Component
@Order(Integer.MAX_VALUE)
public class DemoAuthorizeConfigProvider implements AuthorizeConfigProvider {

```
