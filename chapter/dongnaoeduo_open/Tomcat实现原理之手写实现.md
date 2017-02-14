# Tomcat实现原理手写实现
## 简单版
```java
 public static void main(String[] args) {
        ServerSocket ss = null;
        Socket accept = null;
        try {
            ss = new ServerSocket(9998);
            while (true) {
                accept = ss.accept();
                InputStream is = accept.getInputStream();
                byte[] buf = new byte[1024];
                int len = is.read(buf);
                if (len > 0) {
                    System.out.println(new String(buf, 0, len));
                }

                OutputStream outputStream = accept.getOutputStream();
                outputStream.write("<h1>hello word</h1>".getBytes("UTF-8"));
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                accept.close();
                ss.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```
访问 http://localhost:9998/ 在后台输出如下报文
```java
GET / HTTP/1.1
Host: localhost:9998
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate, sdch, br
Accept-Language: zh-CN,zh;q=0.8
Cookie: bdshare_firstime=1461989447316; pgv_pvid=9782444383; Hm_lvt_7b48cb8bcd42dbe89f577014c9221e43=1473862936,1475644576; _ga=GA1.1.1848041501.1484151117

```

可以看到上面输入流是客户端请求的信息，输出流是服务端响应的信息。那么进一步抽象。

## tomcat 简单版
```java

```