# 传统BIO编程
也就是传统的sock编程。同步阻塞

**服务端**
```java
public class TimeServer {
public static void main(String[] args) throws IOException {
int port = 8086;
ServerSocket server = null;
try {
server = new ServerSocket(port);

System.out.println("服务已启动，端口：" + port);
Socket socket = null;
while (true) {
socket = server.accept(); // 通道链接上后，不手动关闭就会一直存在（长连接）
Socket finalSocket = socket;
new Thread(() -> {
try {
BufferedReader in = new BufferedReader(new InputStreamReader(finalSocket.getInputStream()));
PrintWriter out = new PrintWriter(new OutputStreamWriter(finalSocket.getOutputStream()), true);
String body = null;
while (true) {
body = in.readLine(); // 通过包装后，可以一行一行读取，如果下一行没有数据，则会阻塞，直到有数据为止
System.out.println("== 收到信息：" + body);
if (body == null) {
break;
}
System.out.println("当前时间：" + new Date());
// 数据写出，通过包装后，一行一行的写，必须换行，否则消息不会发出
out.println("当前时间：" + new Date());
}
System.out.println("通信结束");
} catch (IOException e) {
e.printStackTrace();
} finally {
try {
finalSocket.close();
} catch (IOException e) {
e.printStackTrace();
}
}
}).start();
}
} finally {
if (server != null) {
server.close();
}
}
}
}
```

**客户端：**
```java
public class TimeClient {
public static void main(String[] args) throws IOException {
int port = 8086;
Socket socket = new Socket("127.0.0.1", port);
BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream(), "UTF-8"));
PrintWriter out = new PrintWriter(new OutputStreamWriter(socket.getOutputStream(), "UTF-8"), true);
out.println("你好");
String body = null;
while (true) {
body = in.readLine();
if (body == null) {
break;
}
out.println("x");
System.out.println("收到响应消息：" + body);
}
System.out.println("通信结束");
}
}
```

运行以上程序，两边的控制台将会类似无限对话的方式打印消息。

由此可见，由sock连接的通道是长连接，另外一方不主动断开链接，那么将会一直保持通信。

在浏览器中直接访问该地址原理也是一样的。我猜想，浏览器只是发一次请求。如果后台不主动中断链接，那么将会一直阻塞。