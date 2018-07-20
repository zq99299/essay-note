![![](assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180720_134730.pn](assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180720_134730.png)g)# restdocs详细教程-进阶篇2

本节内容：

* 对请求和响应都是json数据的文档输出
* 对文档的请求和响应进行美化处理

## 对请求和响应都是json数据的文档输出

先来看一张效果图
![](/assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180720_133809.png)

编写接受 json串的 controller

```java
    /**
     * 请求和响应都是json串的示例
     * @param book
     * @return
     */
    @PostMapping("/fun4")
    public Book fun4(@RequestBody Book book) {
        book.setAuthors(new String[]{"mrcode", "张4丰", "张5丰"});
        return book;
    }
```

编写测试用例

```java
    /**
     * 增加请求参数 和 响应自定义pojo对象; 对post参数增加请求响应字段的描述
     * 对应的官网文档地址
     * https://docs.spring.io/spring-restdocs/docs/2.0.2.RELEASE/reference/html5/#documenting-your-api-request-response-payloads
     * @throws Exception
     */
    @Test
    public void fun4() throws Exception {
        Book book = new Book();
        book.setName("《mrcode spring resdocs进阶篇》");
        book.setPrice(13.2);
        mockMvc.perform(
                MockMvcRequestBuilders.post("/fun4")
                        .contentType(MediaType.APPLICATION_JSON)
                        // 增加 依赖  compile 'com.alibaba:fastjson:1.2.47'
                        // 对对象进行序列化成json串
                        .content(JSON.toJSONString(book))
        )
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andDo(MockMvcRestDocumentation
                               .document("fun4",
                                         // 这里对于fun3_1中的地方来说
                                         // 只是修改成了 与 响应一样的字段api
                                         PayloadDocumentation.requestFields(
                                                 // 如果有嵌套对象的话。在文档中也说得很清楚了
                                                 // 使用 aa.b 来指定
                                                 PayloadDocumentation.fieldWithPath("name").description("书籍名称"),
                                                 PayloadDocumentation.fieldWithPath("price").description("书籍价格")),
                                         PayloadDocumentation.responseFields(
                                                 PayloadDocumentation.fieldWithPath("name").description("书籍名称"),
                                                 PayloadDocumentation.fieldWithPath("price").description("书籍价格"),
                                                 PayloadDocumentation.fieldWithPath("authors").description("书籍价格")
                                         )
                               )
                );
    }
```

src/docs/asciidoc/fun4.adoc 中和 之前的 fun3_1.adoc 的类似，变化的是一个引用文件，request-fields.adoc 因为这里请求是用的json串了，不再是requestParameters了；
```
== fun4 API

.AP地址

include::{snippets}/fun4/curl-request.adoc[]

.HTTP request

include::{snippets}/fun4/http-request.adoc[]

.Request fields
include::{snippets}/fun4/request-fields.adoc[]

.HTTP response

include::{snippets}/fun4/http-response.adoc[]

.Response fields
include::{snippets}/fun3_1/response-fields.adoc[]
```

有没有发现效果图里面的json串没有格式化，这让一个强迫症的人很伤心

## 对文档的请求和响应进行美化处理

![](/assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180720_134730.png)


这个配置就很简单了,使用预处理器；

```java
    @Before
    public void setUp() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.context)
                // 这一段配置是 https://docs.spring.io/spring-restdocs/docs/current/reference/html5/ 官网中的配置
                // 不需要生成文档的话 不用配置该项
                .apply(documentationConfiguration(this.restDocumentation)
                               // 预处理器，这里设置的是对请求参数和响应参数进行美化输出
                               // https://docs.spring.io/spring-restdocs/docs/2.0.2.RELEASE/reference/html5/#customizing-requests-and-responses-preprocessors-pretty-print
                               // 预处理器的使用 在上面地址文档往上一点点
                               .operationPreprocessors()  // 预处理器
                               .withRequestDefaults(prettyPrint())
                               .withResponseDefaults(prettyPrint())  // 格式化请求或响应参数，比如json串
                )
                .build();
    }
```

增加预处理器后，重新执行fun4的测试用例，和 插件转换命令 ，就ok了。 
如果发现没有效果。很有可能是缓存问题。手动把生成的相关文件删除，再尝试就能看到效果了；


然后又发现一个问题，json串没有语法高亮效果

## 为json串加上语法高亮效果

这个配置很简单全是asciidoc的语法

src/docs/asciidoc/api_list.adoc 中在顶部增加配置

```
= API列表
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toc-title: 目录
:toclevels: 4
:sectlinks:
```

英文好的应该能看明白了；
* `:source-highlighter: highlightjs` 语法高亮
* `:sectlinks:` 为1，2，3 级别标题增加锚点效果，熟悉markdown的人应该知道就是 h1-h6

其他的配置含义官网有详细说明： https://asciidoctor.org/docs/user-manual/#attributes
 

