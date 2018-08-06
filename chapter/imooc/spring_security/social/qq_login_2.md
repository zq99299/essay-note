# QQ登录

上一章节完成了 ServiceProvider的功能，这一节完成应用内部的需要做的一些功能

>注意看这个官网文档： https://docs.spring.io/spring-social/docs/1.1.x/
> 由于在spring-boot-autoconfigure-2.0.4.RELEASE.jar没有对 social的自动配置了
> 所以我搞这节课的连通流程花费了5个小时，最后认证查看官网文档的说明才跑起来


## 实现 ConnectionFactory
```
package cn.mrcode.imooc.springsecurity.securitycore.qq.connet;

import cn.mrcode.imooc.springsecurity.securitycore.qq.api.QQ;
import org.springframework.social.connect.Connection;
import org.springframework.social.connect.support.OAuth2ConnectionFactory;
import org.springframework.social.oauth2.GenericOAuth2ConnectionFactory;

/**
 * qq
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/6 9:02
 * @see GenericOAuth2ConnectionFactory 模仿这个来写
 */
public class QQOAuth2ConnectionFactory extends OAuth2ConnectionFactory<QQ> {
    /**
     * 唯一的构造函数，需要
     * Create a {@link OAuth2ConnectionFactory}.
     * @param providerId 服务商id；自定义字符串；也是后面添加social的过滤，过滤器帮我们拦截的url其中的某一段地址
     *                   on} interface.
     */
    public QQOAuth2ConnectionFactory(String providerId, String appid, String secret) {
      // 传递进来是因为使用该服务的地方才知道  这些参数是什么
        /**
         * serviceProvider 用于执行授权流和获取本机服务API实例的ServiceProvider模型
         * apiAdapter      适配器，用于将不同服务提供商的个性化用户信息映射到 {@link Connection}
         */
        super(providerId, new QQServiceProvider(appid, secret), new QQApiAdapter());
    }
}
```
这里需要提供一个 ApiAdapter

## QQApiAdapter

```java
package cn.mrcode.imooc.springsecurity.securitycore.qq.connet;

import cn.mrcode.imooc.springsecurity.securitycore.qq.api.QQ;
import cn.mrcode.imooc.springsecurity.securitycore.qq.api.QQUserInfo;
import org.springframework.social.connect.ApiAdapter;
import org.springframework.social.connect.Connection;
import org.springframework.social.connect.ConnectionValues;
import org.springframework.social.connect.UserProfile;

/**
 * 适配器，用于将不同服务提供商的个性化用户信息映射到 {@link Connection}
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/6 9:10
 */
public class QQApiAdapter implements ApiAdapter<QQ> {
    @Override
    public boolean test(QQ api) {
        // 测试服务是否可用
        return true;
    }

    @Override
    public void setConnectionValues(QQ api, ConnectionValues values) {
        QQUserInfo userInfo = api.getUserInfo();
        values.setDisplayName(userInfo.getNickname());
        values.setImageUrl(userInfo.getFigureurl_qq_1());
        values.setProfileUrl(null); // 主页地址，像微博一般有主页地址
        // 服务提供商返回的该user的openid
        // 一般来说这个openid是和你的开发账户也就是appid绑定的
        values.setProviderUserId(userInfo.getOpenId());
    }

    @Override
    public UserProfile fetchUserProfile(QQ api) {
        // 暂时不知道有什么用处
        return UserProfile.EMPTY;
    }

    @Override
    public void updateStatus(QQ api, String message) {
        // 应该是退出的状态操作。
    }
}

```

## 开启并配置串联之前写的功能组件
```java
/**
 *
 */
package cn.mrcode.imooc.springsecurity.securitycore.qq;

import cn.mrcode.imooc.springsecurity.securitycore.qq.config.QQAutoConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.security.crypto.encrypt.Encryptors;
import org.springframework.social.UserIdSource;
import org.springframework.social.config.annotation.ConnectionFactoryConfigurer;
import org.springframework.social.config.annotation.EnableSocial;
import org.springframework.social.config.annotation.SocialConfigurerAdapter;
import org.springframework.social.connect.ConnectionFactoryLocator;
import org.springframework.social.connect.ConnectionRepository;
import org.springframework.social.connect.UsersConnectionRepository;
import org.springframework.social.connect.jdbc.JdbcUsersConnectionRepository;
import org.springframework.social.connect.web.ConnectController;
import org.springframework.social.security.AuthenticationNameUserIdSource;
import org.springframework.social.security.SpringSocialConfigurer;

import javax.sql.DataSource;

/**
 * @author zhailiang
 */
@Configuration
@EnableSocial
public class SocialConfig extends SocialConfigurerAdapter {

    @Autowired
    private DataSource dataSource;

    @Override
    public UsersConnectionRepository getUsersConnectionRepository(ConnectionFactoryLocator connectionFactoryLocator) {
        JdbcUsersConnectionRepository repository = new JdbcUsersConnectionRepository(dataSource, connectionFactoryLocator, Encryptors.noOpText());
        // 指定表前缀，后缀是固定的，在JdbcUsersConnectionRepository所在位置
        repository.setTablePrefix("imooc_");
        return repository;
    }

    @Override
    public UserIdSource getUserIdSource() {
        return new AuthenticationNameUserIdSource();
    }

    @Bean
    public SpringSocialConfigurer imoocSocialSecurityConfig() {
        // 默认配置类，进行组件的组装
        // 包括了过滤器SocialAuthenticationFilter 添加到security过滤链中
        SpringSocialConfigurer springSocialConfigurer = new SpringSocialConfigurer();
        return springSocialConfigurer;
    }

    //https://docs.spring.io/spring-social/docs/1.1.x-SNAPSHOT/reference/htmlsingle/#creating-connections-with-connectcontroller
    // 这个在目前阶段不是必须的，
    // 之前不知道为什么就是没有响应
    // 可以暂时忽略该配置
    @Bean
    public ConnectController connectController(
            ConnectionFactoryLocator connectionFactoryLocator,
            ConnectionRepository connectionRepository) {
        return new ConnectController(connectionFactoryLocator, connectionRepository);
    }
}

```

表创建的sql在 JdbcUsersConnectionRepository类所在位置
```sql
-- This SQL contains a "create table" that can be used to create a table that JdbcUsersConnectionRepository can persist
-- connection in. It is, however, not to be assumed to be production-ready, all-purpose SQL. It is merely representative
-- of the kind of table that JdbcUsersConnectionRepository works with. The table and column names, as well as the general
-- column types, are what is important. Specific column types and sizes that work may vary across database vendors and
-- the required sizes may vary across API providers.

create table UserConnection (userId varchar(255) not null,
	providerId varchar(255) not null,
	providerUserId varchar(255),
	rank int not null,
	displayName varchar(255),
	profileUrl varchar(512),
	imageUrl varchar(512),
	accessToken varchar(512) not null,
	secret varchar(512),
	refreshToken varchar(512),
	expireTime bigint,
	primary key (userId, providerId, providerUserId));
create unique index UserConnectionRank on UserConnection(userId, providerId, rank);
```

在 SpringSocialConfigurer 中需要注入 SocialUserDetailsService，之前我们有写好的，改造一下

```java
package cn.mrcode.imooc.springsecurity.securitybrowser;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.social.security.SocialUser;
import org.springframework.social.security.SocialUserDetails;
import org.springframework.social.security.SocialUserDetailsService;
import org.springframework.stereotype.Component;

/**
 * ${desc}
 * @author zhuqiang
 * @version 1.0.1 2018/8/3 9:16
 * @date 2018/8/3 9:16
 * @since 1.0
 */
// 自定义数据源来获取数据
// 这里只要是存在一个自定义的 UserDetailsService ，那么security将会使用该实例进行配置
@Component
public class MyUserDetailsService implements UserDetailsService, SocialUserDetailsService {
    Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private PasswordEncoder passwordEncoder;

    // 可以从任何地方获取数据
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 根据用户名查找用户信息
        logger.info("登录用户名:{}", username);
        // 写死一个密码，赋予一个admin权限
//        User admin = new User(username, "{noop}123456",
//                              AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
        return getUserDetails(username);
    }


    @Override
    public SocialUserDetails loadUserByUserId(String userId) throws UsernameNotFoundException {
        logger.info("登录用户名:{}", userId);
        return getUserDetails(userId);
    }

    private SocialUser getUserDetails(String username) {
        String password = passwordEncoder.encode("123456");
        logger.info("数据库密码{}", password);
        SocialUser admin = new SocialUser(username,
//                              "{noop}123456",
                password,
                true, true, true, true,
                AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
        return admin;
    }
}

```

## 提供qq服务的配置
这个单独拿出来来。方便自动配置和切换

```java
/**
 *
 */
package cn.mrcode.imooc.springsecurity.securitycore.properties;

/**
 * 没有默认值；由使用方注入
 * @author zhailiang
 */
public class QQProperties {
    /**
     * Application id.
     */
    private String appId;

    /**
     * Application secret.
     */
    private String appSecret;
    private String providerId = "qq";
```

```java
/**
 *
 */
package cn.mrcode.imooc.springsecurity.securitycore.properties;

/**
 * @author zhailiang
 *
 */
public class SocialProperties {

	private QQProperties qq = new QQProperties();
```

```java
package cn.mrcode.imooc.springsecurity.securitycore.properties;

import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 * ${desc}
 * @author zhuqiang
 * @version 1.0.1 2018/8/3 15:28
 * @date 2018/8/3 15:28
 * @since 1.0
 */
@ConfigurationProperties(prefix = "imooc.security")
public class SecurityProperties {
    /** imooc.security.browser 路径下的配置会被映射到该配置类中 */
    private BrowserProperties browser = new BrowserProperties();
    private ValidateCodeProperties code = new ValidateCodeProperties();
    private SocialProperties social = new SocialProperties();
```

上面的代码是为了提供配置功能，和之前这些配置一样的思路

下面的配置是为qq登录提供服务商
```java
package cn.mrcode.imooc.springsecurity.securitycore.qq.config;

import cn.mrcode.imooc.springsecurity.securitycore.properties.QQProperties;
import cn.mrcode.imooc.springsecurity.securitycore.properties.SecurityProperties;
import cn.mrcode.imooc.springsecurity.securitycore.qq.connet.QQOAuth2ConnectionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.social.config.annotation.ConnectionFactoryConfigurer;
import org.springframework.social.config.annotation.SocialConfigurerAdapter;
import org.springframework.social.connect.ConnectionFactory;

/**
 * autoconfigure2.04中已经不存在social的自动配置类了
 * org.springframework.boot.autoconfigure.social.SocialAutoConfigurerAdapter
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/6 9:20
 */
@Configuration
// 当配置了app-id的时候才启用
@ConditionalOnProperty(prefix = "imooc.security.social.qq", name = "app-id")
public class QQAutoConfig extends SocialConfigurerAdapter {
    @Autowired
    private SecurityProperties securityProperties;

    @Override
    public void addConnectionFactories(ConnectionFactoryConfigurer configurer,
                                       Environment environment) {
        configurer.addConnectionFactory(createConnectionFactory());
    }

    public ConnectionFactory<?> createConnectionFactory() {
        QQProperties qq = securityProperties.getSocial().getQq();
        return new QQOAuth2ConnectionFactory(qq.getProviderId(), qq.getAppId(), qq.getAppSecret());
    }

    // 后补：做到处理注册逻辑的时候发现的一个bug：登录完成后，数据库没有数据，但是再次登录却不用注册了
    // 就怀疑是否是在内存中存储了。结果果然发现这里父类的内存ConnectionRepository覆盖了SocialConfig中配置的jdbcConnectionRepository
    // 这里需要返回null，否则会返回内存的 ConnectionRepository
  @Override
  public UsersConnectionRepository getUsersConnectionRepository(ConnectionFactoryLocator connectionFactoryLocator) {
      return null;
  }
}

```

浏览器项目中的配置，需要使用apply把开启social的配置文件加入

```java
cn.mrcode.imooc.springsecurity.securitybrowser.BrowserSecurityConfig

// 这里目前注入的其实就是 之前写的开启social的配置类SocialConfig
@Autowired
  private SpringSocialConfigurer imoocSocialSecurityConfig;

  .apply(imoocSocialSecurityConfig)
```

最后注意把 "/auth/*" 路径放行；

## 页面提供qq登录地址

```html
<h3>社交登录</h3>
<!--不支持get请求-->
<form action="/auth/qq" method="post">
    <button type="submit">QQ登录</button>
</form>
```
