# 通信应用-下

由于大多都是简单的业务代码，而且视频很短，不贴代码，不敲代码了。 后面都只记录配置和思路相关的东西

本页内容
* 完成下单服务Feign
* 项目改造成多模块
* 同步or异步
* RabbitMQ的安装
* 微服务，Docker和DevOps

## 完成下单服务Feign

上一章节讲了服务间怎么通信，现在可以把之前下单接口完成了；

商品服务提供，订单服务使用Feign方式调用，目前在这里是没有事务概念的。

```
// TODO: 2018/8/16 查询商品信息（调用商品服务）
// TODO: 2018/8/16  计算总价  本地计算就ok
// TODO: 2018/8/16  扣库存（调用商品服务）
```

## 项目改造成多模块

改造成多模块的原因是：

1. 在编写代码的过程中，已经发现了 需要自己去写商品信息类，然而这些是服务方可以提供一个jar包出来的；
2. 定义的接口，也是可以由服务方提供出来的，如果都是用一套技术栈的话

如下所示：

1. 提供一个client模块 和 一个 common模块
2. ProductInfoOutput 和 DecreaseStockInput 入参和出参对象只是java bean,放在common中，供客户端和服务提供商共同引用使用
3. ProductClient 放在 client模块，表示是可以提供给你这么多的服务，可以直接扫描调用；

```java
@FeignClient(name = "product")
public interface ProductClient {
    // 商品信息查询服务
    @PostMapping("/product/listForOrder")
    List<ProductInfoOutput> listForOrder(@RequestBody List<String> productIdList);

    // 扣库存服务
    @PostMapping("/product/decreaseStock")
    void decreaseStock(@RequestBody List<DecreaseStockInput> decreaseStockInputList);
```

在服务端提供的公用类和公用服务接口的方式，大部分场景下都能满足需求了。自己的服务相互调用的话，这种方案还是比较不错的
