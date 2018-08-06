# apache.commons 技巧记录

## org.apache.commons.lang3

* RandomStringUtils 随机字符串工具类
* StringUtils 字符串工具类

  ```java
  // 拿匹配结构的后缀，返回结果 sms
  StringUtils.substringAfter("/code/sms", "/code/");
  --------------------------------------------------------------
  获取匹配结果的前缀
  // ImageCodeProcessor 类名
  // 结果：Image
  // 使用场景：只用持有者管理所有实现子类的时候，可以拿到前缀，然后根据前缀拿到相关的枚举信息
  StringUtils.substringBefore(getClass().getSimpleName(), "CodeProcessor");
  --------------------------------------------------------------
  提供类似与正则表达式的字符串截取
  // callback( {"client_id":"YOUR_APPID","openid":"YOUR_OPENID"} );
  // 返回 YOUR_OPENID
  StringUtils.substringBetween("\"openid\":", "\"}");

  ---------------------------------------------------------------
  替代传统多次按等号分割，但是又只想获取值的情况
  // access_token=FE04*******CCE2&expires_in=7776000&refresh_token=88E4*********BE14
  String[] items = StringUtils.splitByWholeSeparatorPreserveAllTokens(responseStr, "&");
  String accessToken = StringUtils.substringAfterLast(items[0], "=");
  --------------------------------------------------------------
  ```
