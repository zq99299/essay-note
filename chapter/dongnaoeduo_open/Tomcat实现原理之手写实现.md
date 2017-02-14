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
public class Service {
    public static void main(String[] args) {
        ServerSocket ss = null;
        Socket accept = null;
        try {
            ss = new ServerSocket(9999);
            while (true) {
                accept = ss.accept();
                HttpRequest httpRequest = new HttpRequest(accept.getInputStream());
                HttpResponse httpResponse = new HttpResponse(accept.getOutputStream());

                // 判断是否是静态资源
                String uri = httpRequest.getUri();
                if (isStatic(uri)) {
                    httpResponse.writeFile(Service.class.getResource(uri).getFile());
                } else {
                    httpResponse.writeFile(Service.class.getResource("/404.html").getFile());
                }
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

    private static boolean isStatic(String uri) {
        if (uri == null) {
            return false;
        }
        String[] suffixs = {"jpg", "html"};
        for (String suffix : suffixs) {
            if (uri.endsWith(suffix)) {
                return true;
            }
        }
        return false;
    }
}
public class HttpRequest {
    private String method;// 请求方法
    private String uri;//请求的URI地址  在HTTP请求的第一行的请求方法后面
    private String host;//请求的主机信息

    public HttpRequest(InputStream inputStream) {
        this.parse(inputStream);
    }

    private void parse(InputStream is) {
        BufferedReader br = new BufferedReader(new InputStreamReader(is, Charset.forName("UTF-8")));
        String line = null;
        try {
            // 要注意：sock 的输入流和普通的文件流不同，它不知道客户端是否还在输出信息
            // 如果不主动放弃读，继续往下读的话，就会阻塞
            while ((line = br.readLine()) != null) {
                // 因为http协议有报文头，空行 作为结束行
                if(StringUtils.isBlank(line)){
                    break;
                }
                doParse(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } /*finally {
            try {
                is.close();  // 不能关闭，关闭流会导致当前链接被关闭，外部不能继续操作 java.net.SocketException: Socket is closed
            } catch (IOException e) {
                e.printStackTrace();
            }
        }*/
    }

    /**
     * 开始具体解析每一行的报头内容
     * @param line
     */
    private void doParse(String line) {
        // 简单提取
        if (line.startsWith("GET")) {
            this.setMethod("GET");
            this.setUri(line.substring(4, line.lastIndexOf("HTTP") -1));
        } else if (line.startsWith("Host")) {
            this.setHost(line.substring(6, line.length()));
        }
    }

    public String getMethod() {
        return method;
    }

    public void setMethod(String method) {
        this.method = method;
    }

    public String getUri() {
        return uri;
    }

    public void setUri(String uri) {
        this.uri = uri;
    }

    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }
}
public class HttpResponse {
    private OutputStream os = null;

    public HttpResponse(OutputStream os) {
        this.os = os;
    }

    /**
     * 响应静态文件
     * @param path
     */
    public void writeFile(String path) throws IOException {
        sendResponseHeaders(path);
        FileInputStream is = new FileInputStream(path);
        int len = 0;
        byte[] buf = new byte[1024];
        while ((len = is.read(buf)) != -1) {
            os.write(buf, 0, len);
        }
        os.flush();
        is.close();
        os.close();
    }

    /**
     * 模拟返回响应头
     * @param path
     */
    private void sendResponseHeaders(String path) {
        File file = new File(path);
        PrintStream writer = new PrintStream(os);

        writer.println("HTTP/1.0 200 OK");// 返回应答消息,并结束应答
        writer.println("Content-Length:" + file.length());// 返回内容字节数
        if (path.endsWith(".html")) {
            writer.println("Content-Type:text/html;charset=UTF-8");
        }

        writer.println();// 根据 HTTP 协议, 空行将结束头信息
    }
}

```

在resources中增加几个html和jpg图片，启动后可访问测试。

### 总结
整个过程其实就是在对http协议的请求头和响应头做解析