# 使用WireMock快速伪造restful服务

在后端没有开发好的时候，使用wireMock快速伪造服务；

## 为什么要伪造服务？
主要是用于多端并行开发，前段可能有pc、app，微信多端点，遇到这种情况，先伪造服务的确是能节省很多成本；
当然如果很耗费时间的话，肯定就得不偿失了；
到时候等后端写好之后，前段切换连接就行了

## wireMock安装

>官网 http://wiremock.org/
> 下载页面 http://wiremock.org/docs/running-standalone/
下载jar包，然后命令运行
```
$ java -jar wiremock-standalone-2.18.0.jar --port 9999
```

打开 `http://localhost:9999/` 现在会报错，因为没有什么东西；

运行之后会在当前目录下产生两个文件夹,目前我也不知道是干什么的。先跟着学习
```
|-__files
|-mappings
```

## 编写wireMock需要的数据
> http://wiremock.org/docs/getting-started/  
官网的入门页面会教你怎么添加依赖和编写测试生成数据

截止笔记记录时最新版本；
```
testCompile "com.github.tomakehurst:wiremock:2.18.0"
```
官网介绍的是JUnit 4.x的编写，视频中介绍的是 静态方法编写。在src/main中编写的话，依赖需要换成 compile

## wireMock数据hello word
```java
package com.example.demo.web.wiremock;

import static com.github.tomakehurst.wiremock.client.WireMock.*;

/**
 * @author : zhuqiang
 * @version : V1.0
 * @date : 2018/8/2 23:12
 */
public class WiremockServer {
    public static void main(String[] args) {
        configureFor(9999); // 独立服务的端口，在本机就不用写ip了
        removeAllMappings(); // 清除所有的配置，因为每次更新都需要重新写入配置信息
        // 编写一个测试桩
        // get请求，严格匹配一个url地址
        stubFor(get(urlPathEqualTo("/order/1"))
                // 定义该地址返回的数据和http状态
                .willReturn(aResponse().withBody("{\"id\":\"1\"}").withStatus(200))
        );
    }
}
```

运行之后，就可以访问`http://localhost:9999/order/1` 查看返回的信息

## 重构优化wireMock的测试桩

把响应json串可以提取到文件中。
resources/mock/01.json
```json
{
  "id": "1",
  "name": "产品名称"
}
```

改造后的代码
```java
public class WiremockServer {
    public static void main(String[] args) throws IOException {
        configureFor(9999); // 独立服务的端口，在本机就不用写ip了
        removeAllMappings(); // 清除所有的配置，因为每次更新都需要重新写入配置信息
        mock("/order/1", "01");
    }

    private static void mock(String url, String fileName) throws IOException {
        // 编写一个测试桩
        // get请求，严格匹配一个url地址
        ClassPathResource json01 = new ClassPathResource("mock/" + fileName + ".json");
        // https://mvnrepository.com/artifact/commons-io/commons-io
//        compile group: 'commons-io', name: 'commons-io', version: '2.6'
        String body = org.apache.commons.io.FileUtils.readFileToString(json01.getFile(), "utf-8");
        stubFor(get(urlPathEqualTo(url))
                // 定义该地址返回的数据和http状态
                .willReturn(aResponse()
                        .withBody(body) // 返回内容
                        .withStatus(200) // http状态码
                        // 添加响应头，防止中文乱码
                        .withHeader("Content-Type", "application/json;charset=UTF-8"))
        );
    }
}
```

## 总结
简单使用的话，非常的方便，需要研究下具体的功能，介绍说很强大；
