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

