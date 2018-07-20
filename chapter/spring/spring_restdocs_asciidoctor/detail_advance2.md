# restdocs详细教程-进阶篇2

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

