# rest服处理文件上传
现在的文件上传服务基本上都是先上传，后提交路径

## 测试用例

```java
@Test
public void whenFileUploadSuccess() throws Exception {
    // v5.0+ fileUpLoad方法已经过时了
    String file = mockMvc.perform(multipart("/file")
                                          .file(new MockMultipartFile("file", "test.txt", "multipart/form-data", "hello upload".getBytes())))
            .andExpect(status().isOk())
            .andReturn().getResponse().getContentAsString();
    System.out.println(file);
}

输出
{"absolutePath":"G:\\dev\\project\\mrcode\\example\\imooc\\spring-security\\security-demo\\fileUploadTest.txt"}
```

文件上传服务
```java
package com.example.demo.web.controller;

import com.example.demo.dto.FileInfo;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.IOException;

@RestController
@RequestMapping("/file")
public class FileController {
    @PostMapping
    public FileInfo upload(MultipartFile file) throws IOException {
        System.out.println(file.getName());
        System.out.println(file.getOriginalFilename());
        System.out.println(file.getSize());

        File localFile = new File("fileUploadTest.txt");
        file.transferTo(localFile);
        return new FileInfo(localFile.getAbsolutePath());
    }
}
```

## 文件下载服务

下载服务好像在测试用例里面没有看到怎么写的；浏览器访问该路径，文件会被下载

```java
@GetMapping("/{id}")
public void download(@PathVariable String id, HttpServletRequest request, HttpServletResponse response) {
    try (FileInputStream inputStream = new FileInputStream(new File("G:\\dev\\project\\mrcode\\example\\imooc\\spring-security\\security-demo", id));
         ServletOutputStream outputStream = response.getOutputStream();
    ) {
        // 声明响应类型
        response.setContentType("application/x-download");
        // 下载的文件名称
        response.addHeader("Content-Disposition", "attachment;filename-test.txt");
        IOUtils.copy(inputStream, outputStream);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
