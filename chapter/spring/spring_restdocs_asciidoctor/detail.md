# restdocs详细教程

## spring boot2 项目搭建

新建spring boot2 项目，依赖选择如下图
![](/assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180720_090426.png)

从ui选择创建spring boot 2 项目后，build.gradle 如下所示的初始化配置
```
buildscript {
	ext {
		springBootVersion = '2.0.3.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'cn.mrcode.example.spring.restdocs'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile('org.springframework.boot:spring-boot-starter-web')
	testCompile('org.springframework.boot:spring-boot-starter-test')
	testCompile('org.springframework.restdocs:spring-restdocs-mockmvc')
}
```

## 编写HelloWordDocsController

```java
    @GetMapping("/fun1")
    public List<Integer> fun1() {
        return Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);
    }
```

## 编写HelloWordDocsControllerTest测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloWordDocsControllerTest {
    @Rule
    public final JUnitRestDocumentation restDocumentation = new JUnitRestDocumentation();
    @Autowired
    private WebApplicationContext context;

    private MockMvc mockMvc;

    @Before
    public void setUp() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.context)
                // 这一段配置是 https://docs.spring.io/spring-restdocs/docs/current/reference/html5/ 官网中的配置
                // 不需要生成文档的话 不用配置该项
                .apply(documentationConfiguration(this.restDocumentation))
                .build();
    }

    @Test
    public void fun1() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/fun1")) // 请求
                .andExpect(MockMvcResultMatchers.status().isOk()) // 断言HTTP状态为200，否则异常
                .andDo(MockMvcRestDocumentation  // 增加文档；原理就是收集一些请求响应数据按照asciidoctor语法生成“.adoc”文件；
                               .document("fun1"));   // 这个api就是专为生成asciidoctor的配置api；更详细的配置可以参考他的官网
    }
```

运行 `fun1()` 测试方法后，会默认在build/generated-snippets中生成fun1文档目录和响应的代码片断，如下图

![](/assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180720_093241.png)

这里生成的代码片断，也就是fun1目录下的.adoc文件，打开看的话都是很简单的 asciidoctor 语法；可以看到之前说的 spring提供的api就是为了抓取到相应的数据，然后按照 asciidoctor 语法生成文件；

## 使用插件把这个代码片断转成 html文件




