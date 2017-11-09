# Google Protobuf 编解码

在业界非常流程，很多商业项目都选用它作为编解码工具，优点：
1. 在谷歌内部长期使用，昌平成熟度高
2. 跨语言、支持多种语言，包括`C++`,`Java`和`Python`
3. 编码后的消息更小，更加利于存储和传输
4. 编解码的性能非常高
5. 支持不同协议版本的前向兼容
6. 支持定义可选和必选字段

本章主要内容包括：
- `Protobuf`的入门
- 开发支持`Protobuf`的`Netty`服务端
- 开发支持`Protobuf`的`Netty`客户端
- 运行基于`Netty`开发的`Protobuf`程序

# `Protobuf`的入门
## 开发环境

在以下网站
https://github.com/google/protobuf/releases/tag/v3.0.0
中找到 `protoc-3.0.0-win32.zip`,解压之后：会得到`bin\protoc.exe`

通过该命令生成类；

`注:` 需要将`protoc.exe` 所在目录路径添加到path系统环境变量中，方便在任何地方都能访问`protoc`命令

## 编写一个.protoc文件
Idea中有相应的插件：
Plugin homepage
http://github.com/nnmatveev/idea-plugin-protobuf
和 Google Protobuf

插件也不知道怎么使用，但是有语法高亮，却没有语法提示

**SubscribeReq.proto**

```java
syntax = "proto2"; // 2和3的语法感觉很多不一样吧。如 required 在3中不允许。还是先按照书上使用2的语法(因为之前下载的3的命令版本所以这里不写语法的话，会提示你写上语法版本)
package netty;
option java_package = "cn.mrcode.d20170227nettycodec.protobuf";
option java_outer_classname = "SubscribeReqProto"; // 这里的类名不能和 定义的名称一致

message SubscribeReq {
    required int32 subReqId = 1;
    required string userName = 2;
    required string productName = 3;
    required string address = 4;
}
```

**SubscribeResp.proto**
```java
syntax = "proto2"; 
package netty;
option java_package = "cn.mrcode.d20170227nettycodec.protobuf";
option java_outer_classname = "SubscribeRespProto";

message SubscribeResp {
    required int32 subReqId = 1;
    required int32 respCode = 2;
    required string desc = 3;
}
```
编译命令：
```bash
$ protoc --java_out=./ ./SubscribeReq.proto
$ protoc --java_out=./ ./SubscribeResp.proto


语法是：$ protoc --java_out=输出目录 具体的.proto文件

```

生成完类之后，拷贝到相对应的项目中的包路径下。
却发现报错了。缺少依赖。
```groovy
    // https://mvnrepository.com/artifact/com.google.protobuf/protobuf-java
    compile group: 'com.google.protobuf', name: 'protobuf-java', version: '3.2.0'
    
这里同样也引用v3版本的jar。因为使用v3版本的编译器生成的，我发现如果引用v2版本的，还是会报错。
```

## 编码解码开发
写一个测试用例，怎么编码解码。
```java
public class SubscribeReqProtoTest {
    public static void main(String[] args) throws InvalidProtocolBufferException {
        SubscribeReqProto.SubscribeReq req = createSubscribeReq();
        System.out.println("编码前：" + req.toString());

        byte[] bodys = encode(req);
        System.out.println("编码后:" + bodys.length);
        SubscribeReqProto.SubscribeReq afterReq = decode(bodys);
        System.out.println("解码后：" + afterReq.toString());
        System.out.println("是否相等：" + (req == afterReq));
        System.out.println("是否相等：" + (req.equals(afterReq)));

        String str = req.getSubReqId() + req.getUserName() + req.getProductName() + req.getAddress() + "userNamesubReqIdproductNameaddress";
        System.out.println("对比下自己把所有字符相加得到的字节大小：" + str.getBytes().length);

    }

    private static SubscribeReqProto.SubscribeReq decode(byte[] body) throws InvalidProtocolBufferException {
        return SubscribeReqProto.SubscribeReq.parseFrom(body);
    }

    private static byte[] encode(SubscribeReqProto.SubscribeReq req) {
        return req.toByteArray();
    }

    private static SubscribeReqProto.SubscribeReq createSubscribeReq() {
        SubscribeReqProto.SubscribeReq.Builder builder = SubscribeReqProto.SubscribeReq.newBuilder();
        builder.setSubReqId(1);
        builder.setUserName("张小凡");
        builder.setProductName("Star");
        builder.setAddress("青云门");
        return builder.build();
    }
}
```

输出：
```java
编码前：subReqId: 1
userName: "\345\274\240\345\260\217\345\207\241"
productName: "Star"
address: "\351\235\222\344\272\221\351\227\250"

编码后:30
解码后：subReqId: 1
userName: "\345\274\240\345\260\217\345\207\241"
productName: "Star"
address: "\351\235\222\344\272\221\351\227\250"

是否相等：false
是否相等：true
对比下自己把所有字符相加得到的字节大小：57
```

这里可以看到编码后的大小才30，而我们自己手动把所有的值和字段名称放在一起直接变成57. 虽然不知道为什么，但是感觉好厉害的样子。 因为要保留类型等信息。