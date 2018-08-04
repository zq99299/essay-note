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

## 依赖查找
** 依赖查找：** 当有多个子类实现的时候，根据beanNam进行查找需要的子类；
如下图的autowired,声明是一个Map,那么key = beanName、value = 不同的子类实现

** 使用场景: ** 编写基础架子，提供了多种相同功能的不同实现策略的时候
```java
@Autowired
private Map<String, ValidateCodeGenerate> validateCodeGenerates;
```
