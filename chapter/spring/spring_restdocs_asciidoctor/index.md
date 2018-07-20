# spring-restdocs-asciidoctor

> spring官网教程 - 里面有相关构建工具的插件引入方式
> https://docs.spring.io/spring-restdocs/docs/current/reference/html5/
>
> asciidoctor 官网文档：生成的文档侧栏等信息 就是在该文档中找到的
> https://asciidoctor.org/docs/user-manual/#left-or-right-column-layout

我在百度找了很多资料，基本上都是对官网文档的翻译，或则是很古老的版本； 这次是spring boot2版本；

其实原理很简单：

* 使用mockMvc进行测试用例的编写
* 增加文档的配置描述
* 使用插件把文档api生成的.adoc文件转换成 html

> github教程示例项目: https://github.com/zq99299/spring-restdocs-example