# 使用swagger自动生成html文档

本节内容

* 使用swagger自动生成html文档
* 使用WireMock快速伪造restful服务

前后分离并行开发的时候（当然不是一个人从前到后都干那种）；那么提供文档就很有必要了。

光看文档不是那么的直观。伪造服务可能更直观（个人感觉而言，文档详细，自己在postman这种工具中去调用也是一样的）

## 初体验
添加两个依赖

```
// 扫描程序生成文档数据
// http://springfox.github.io/springfox/docs/current/
// https://mvnrepository.com/artifact/io.springfox/springfox-swagger2
compile group: 'io.springfox', name: 'springfox-swagger2', version: '2.9.2'
// 提供可视化界面
// https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui
compile group: 'io.springfox', name: 'springfox-swagger-ui', version: '2.9.2'
```

添加注解开启swagger
```java
@EnableSwagger2
public class DemoApplication {
```

启动程序后，访问：`http://localhost:8080/swagger-ui.html` 就能看到一个界面。里面会显示该程序中所有的的controller断点。并扫描该断点的注解等信息进行分析一些有关断点的信息；

可以点击`Try it out`按钮发起请求，然后在界面上会把响应结果返回；

这样看的确挺方便的；

## 使用注解自定义信息

定义api描述
```java
@ApiOperation(value = "用户查询服务")  // 方法描述
public List<User> query(UserQueryCondition condition) {
```

定义请求字段信息，如果参数是一个对象，则需要在对象字段上添加注解
```java
public class UserQueryCondition {
    @ApiModelProperty(value = "用户名")
    private String username;
```

不是对象的字段描述

```java
@JsonView(User.UserDetailView.class)
public User getInfo(@ApiParam(value = "用户id") @PathVariable String id) {
```

上面是3个常用的注解，其他的官网文档查看；

个人感觉：相对于spring-restdocs-asciidoctor代码入侵太严重，我个人是不太愿意用的；但是的确很方便能看到所有提供的服务；包括spring框架提供的

> 下一节：使用WireMock快速伪造restful服务
