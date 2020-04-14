# spring-restdocs-asciidoctor

*  [spring官网教程](https://docs.spring.io/spring-restdocs/docs/current/reference/html5/)

    里面有相关构建工具的插件引入方式
*  [AsciiDoc 官网文档](https://asciidoctor.org/docs/user-manual/#left-or-right-column-layout)：

    生成的文档侧栏等信息 就是在该文档中找到的
* [ AsciiDoc 语法快速参考](https://asciidoctor.org/docs/asciidoc-syntax-quick-reference)

我在百度找了很多资料，基本上都是对官网文档的翻译，或则是很古老的版本； 这次是spring boot2版本；

其实原理很简单：

* 使用mockMvc进行测试用例的编写
* 增加文档的配置描述
* 使用插件把文档api生成的.adoc文件转换成 html

> github教程示例项目: https://github.com/zq99299/spring-restdocs-example

学习这个的使用有几点别搞混了：

1. asciidoctor 和 markdown 类似，与gitbook更类似，都是需要转换器进行转换
2. 无侵入源代码，只是在测试用例的时候 附加生成文档

## 目录导航

* [restdocs详细教程-入门篇](/chapter/spring/spring_restdocs_asciidoctor/detail.md)
* [restdocs详细教程-进阶篇1](/chapter/spring/spring_restdocs_asciidoctor/detail_advance.md)
* [restdocs详细教程-进阶篇2](/chapter/spring/spring_restdocs_asciidoctor/detail_advance2.md)

[前往 CSDN 地址观看](https://blog.csdn.net/mr_zhuqiang/article/details/81132362)
