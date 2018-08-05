# QQ登录上
根据上面接收的原理，开发思路是：

最终我们是要获取到Connection,

那么就需要使用ConnectionFactory创建

而ConnectionFactory又需要ServiceProvider和ApiAdapter；

倒退流程，走到最后就是先实现Api。

> http://wiki.connect.qq.com/api%E5%88%97%E8%A1%A8

## 开发Api对象

```java
package cn.mrcode.imooc.springsecurity.securitycore.qq.api;

import java.io.IOException;

public interface QQ {
    QQUserInfo getUserInfo() throws IOException;
}
```

```java
import org.springframework.social.oauth2.TokenStrategy;

import java.io.IOException;

/**
 * 构建与qq交互的api实例;完成的是第6步
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/6 0:46
 */
public class QQImpl extends AbstractOAuth2ApiBinding implements QQ {
    // http://wiki.connect.qq.com/%E8%8E%B7%E5%8F%96%E7%94%A8%E6%88%B7openid_oauth2-0
    public final static String URL_GET_OPENID = "https://graph.qq.com/oauth2.0/me?access_token=%s";
    // 父类会自动携带accessToken
    public final static String URL_GET_USER_INFO = "https://graph.qq.com/user/get_user_info?oauth_consumer_key=%s&openid=%s";

    private String appId;
    private String openid;

    private ObjectMapper objectMapper = new ObjectMapper();

    public QQImpl(String accessToken, String appId) {
        // 该语句代码查看继承类的源码得知
        // 默认是把accessToken放入请求头中
        // qqapi的文档确是放在参数中传递的
        super(accessToken, TokenStrategy.ACCESS_TOKEN_PARAMETER);
        this.appId = appId;
        String url = String.format(URL_GET_OPENID, accessToken);
        //callback( {"client_id":"YOUR_APPID","openid":"YOUR_OPENID"} );
        String result = getRestTemplate().getForObject(url, String.class);
        System.out.println(result);
        this.openid = StringUtils.substringBetween("\"openid\":", "}");
    }

    @Override
    public QQUserInfo getUserInfo() throws IOException {
        String url = String.format(URL_GET_USER_INFO, appId, openid);
        String result = getRestTemplate().getForObject(url, String.class);
        System.out.println(result);
        QQUserInfo qqUserInfo = objectMapper.readValue(result, QQUserInfo.class);
        return qqUserInfo;
    }
}

```

```java
package cn.mrcode.imooc.springsecurity.securitycore.qq.api;

/**
 * qq用户信息:
 * http://wiki.connect.qq.com/get_user_info
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/6 0:45
 */
public class QQUserInfo {

    /**
     * 返回码
     */
    private String ret;
    /**
     * 如果ret<0，会有相应的错误信息提示，返回数据全部用UTF-8编码。
     */
    private String msg;
    /**
     *
     */
    private String openId;
    /**
     * 不知道什么东西，文档上没写，但是实际api返回里有。
     */
    private String is_lost;
    /**
     * 省(直辖市)
     */
    private String province;
    /**
     * 市(直辖市区)
     */
    private String city;
    /**
     * 出生年月
     */
    private String year;
    /**
     * 用户在QQ空间的昵称。
     */
    private String nickname;
    /**
     * 大小为30×30像素的QQ空间头像URL。
     */
    private String figureurl;
    /**
     * 大小为50×50像素的QQ空间头像URL。
     */
    private String figureurl_1;
    /**
     * 大小为100×100像素的QQ空间头像URL。
     */
    private String figureurl_2;
    /**
     * 大小为40×40像素的QQ头像URL。
     */
    private String figureurl_qq_1;
    /**
     * 大小为100×100像素的QQ头像URL。需要注意，不是所有的用户都拥有QQ的100×100的头像，但40×40像素则是一定会有。
     */
    private String figureurl_qq_2;
    /**
     * 性别。 如果获取不到则默认返回”男”
     */
    private String gender;
    /**
     * 标识用户是否为黄钻用户（0：不是；1：是）。
     */
    private String is_yellow_vip;
    /**
     * 标识用户是否为黄钻用户（0：不是；1：是）
     */
    private String vip;
    /**
     * 黄钻等级
     */
    private String yellow_vip_level;
    /**
     * 黄钻等级
     */
    private String level;
    /**
     * 标识是否为年费黄钻用户（0：不是； 1：是）
     */
    private String is_yellow_year_vip;

```

## 开发服务提供商

```java
package cn.mrcode.imooc.springsecurity.securitycore.qq.connet;

import cn.mrcode.imooc.springsecurity.securitycore.qq.api.QQ;
import cn.mrcode.imooc.springsecurity.securitycore.qq.api.QQImpl;
import org.springframework.social.oauth2.AbstractOAuth2ServiceProvider;
import org.springframework.social.oauth2.OAuth2ServiceProvider;
import org.springframework.social.oauth2.OAuth2Template;

/**
 * 服务提供商:
 * 官网地址可以获取 authorizeUrl 和 accessTokenUrl
 * http://wiki.connect.qq.com/%E5%BC%80%E5%8F%91%E6%94%BB%E7%95%A5_server-side
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/6 1:20
 */
public class QQServiceProvider extends AbstractOAuth2ServiceProvider<QQ> {
    public static final String authorizeUrl = "https://graph.qq.com/oauth2.0/authorize";
    public static final String accessTokenUrl = "https://graph.qq.com/oauth2.0/token";
    private String appId;

    /**
     * Create a new {@link OAuth2ServiceProvider}.
     */
    public QQServiceProvider(String appId, String secret) {
        // OAuth2Operations 有一个默认实现类，可以使用这个默认实现类
        // oauth2的一个流程服务
        super(new OAuth2Template(appId, secret, authorizeUrl, accessTokenUrl));
    }

    @Override
    public QQ getApi(String accessToken) {
        return new QQImpl(accessToken, appId);
    }
}
```

对于social基本概念和原理中的图上右侧的服务提供商的对象已经开发完成
