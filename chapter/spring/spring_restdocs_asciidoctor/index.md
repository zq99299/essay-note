# spring-restdocs-asciidoctor


这里是一个概述入门文章，在spring boot项目中怎么使用 spring-restdocs-asciidoctor；效果图
![](/assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180719_111106.png)

步骤如下：

1. 使用spring-restdocs-mockmvc 编写controller的测试用例，根据api增加自定义字段描述等个性化信息；会生成代码片断
2. 编写插件组合代码片断的 .adoc 文件
3. 使用插件转成html文件，插件会读取第2步中的文件


> spring官网教程 - 里面有相关构建工具的插件引入方式
> https://docs.spring.io/spring-restdocs/docs/current/reference/html5/
>
> asciidoctor 官网文档：生成的文档侧栏等信息 就是在该文档中找到的
> https://asciidoctor.org/docs/user-manual/#left-or-right-column-layout

我以gradle为例

## 增加依赖

这里的依赖是在一个现有的spring boot2 web项目上增加的配置
```
// rest doc 1  转成html插件
plugins {
    id "org.asciidoctor.convert" version "1.5.3"
}
apply plugin: 'java'

// 注意 org.asciidoctor.convert 插件配置要在 apply语法前面 否则gradle运行报错

repositories {
    mavenCentral()
    // rest doc 2 由于插件貌似不在中心仓库，指向spring官网仓库下载
    maven { url 'https://repo.spring.io/libs-snapshot' }
}
ext {
    // rest doc 3
    springRestdocsVersion = '2.0.2.BUILD-SNAPSHOT'
    // 测试用例api生成的片断代码目录 gradle默认在该目录下
    // 这里配置变量只是为了在后面的插件配置中使用该变量
    snippetsDir = file('build/generated-snippets')
}
dependencies {
    // rest doc 4
    testCompile('org.springframework.restdocs:spring-restdocs-mockmvc')
    asciidoctor "org.springframework.restdocs:spring-restdocs-asciidoctor:${springRestdocsVersion}"
}

// rest doc 5  这个还不知道是啥意思
test {
    useTestNG()
    outputs.dir snippetsDir
}

// rest doc 6 gradle插件命令，默认输出目录在build/asciidoc/html5中生成
// 插件配置
asciidoctor {
    inputs.dir snippetsDir
    dependsOn test
}
```

## api编写和代码片断生成
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloControllerTest {
    @Rule
    public final JUnitRestDocumentation restDocumentation = new JUnitRestDocumentation();
    
    @Autowired
    private WebApplicationContext context;

    private MockMvc mockMvc;

    @Before
    public void setUp() {
        // 官网教程中的 使用 doc 实例程序
        // 这个组装步骤很重要，否则不会生成相应的代码片断
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.context)
                .apply(documentationConfiguration(this.restDocumentation)).build();
    }
    
    @Test
    public void fun1() throws Exception {
        mockMvc.perform(get("/test/fun1"))
                .andExpect(status().isOk())
                .andDo(document("fun1"));  // 最简单的代码片断生成
    }
    @Test
    public void fun2() throws Exception {
        mockMvc.perform(get("/test/fun2"))
                .andExpect(status().isOk())
                .andDo(document("fun2"));
    }
}
```

执行 fun1后 会在  build/generated-snippets/fun1 中生成以下文件信息; 这些就是这个api相关的请求信息，上面代码是默认的配置，所以里面信息很少； 具体的需要待研究api怎么添加

```
curl-request.adoc
http-request.adoc
http-response.adoc
httpie-request.adoc
request-body.adoc
response-body.adoc
```

## 构建 转换插件锁需要的 文件
src/docs/asciidoc/index.adoc

```java
= REST API 文档
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toc-title: 目录
:toclevels: 4
:sectlinks:

== 首页
使用 `GET` 请求来访问首页。
operation::fun1[]

== fun2
operation::fun2[]

== fun3
operation::fun3[]
```

这里的文件是什么名称，执行 gradle asciidoctor 命令后 就会在build/asciidoc/html5中生成相同名称的html文件； 多个文件和自定义格式要研究 asciidoctor 官网文档了；

这里简单说下几个配置语法

* 文档开头 冒号的配置,要生成侧边栏则需要配置一下几个参数，具体参数含义查看官网文档
```
:toc: left
:toc-title: 目录
:toclevels: 4
```
```
== fun2      // 生成一个侧边栏目录标题
operation::fun2[]   // 标识包含build/generated-snippets 目录下 指定名称的 代码包片断
// 这样写的话，默认内容就是所有的生成文件安装默认模版加载展示
// 要自己组织结构 需要自己编写该代码片断的组织文件，百度很多教程就是自定义组织文件的
// 其他详细定制有空再研究
```
