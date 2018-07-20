# restdocs详细教程-进阶篇

本章内容：

1. 进一步了解 spring-restdocs的使用
2. 完成一个常见api文档的排版效果
3. 侧边栏的配置
4. 自定义文档内容的编写


## 实现左侧目录功能

新增 src/docs/asciidoc/fun1_summary.adoc 文件，内容如下

```
= API列表
:toc: left

.fun api列表

operation::fun1[]
```

注意顶部的 `:toc: left` left后面有空格，这个就是 asciidoctor 语法；该配置官网链接
https://asciidoctor.org/docs/user-manual/#setting-attributes-on-a-document

再次运行 gradle asciidoctor 命令；打开 build/asciidoc/html5/fun1_summary.html 文件
![](/assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180720_111635.png)

目录已经出来了，但是效果并不好看。继续配置

## 左侧目录优化
先来看一张效果图
![](/assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180720_112129.png)

这个图和我们的目标差不多了；

* 有侧边目录
* 请求地址
* 请求参数 （该api中没有参数所以没有显示）
* 响应参数

新增两个文件；目录结构如下

```bash
|-- docs
    |- asciidoc
        |- api_list.adoc
        |- fun1.adoc
        |- fun1_summary.adoc
        |- fun1_summary_2.adoc
```

api_list.adoc 入口文件，汇总所有需要到一个html中的配置

```
= API列表
:toc: left

include::./fun1_summary_2.adoc[]
```
include 指令，不用多说，引用了当前目录下的fun2自定义内容版本文件


fun1_summary_2.adoc 对fun1的内容进行定制

```
== fun1 API

=== API地址

include::{snippets}/fun1/curl-request.adoc[]

.HTTP request:

include::{snippets}/fun1/http-request.adoc[]

.HTTP response:

include::{snippets}/fun1/http-response.adoc[]
```

> {snippets} 忘记哪里来的了。好像是 转换插件里面的预定变量。指向了测试用例代码片断所在目录

上面文件内容百分之98的内容都是根据 asciidoctor 官网文档 编写
