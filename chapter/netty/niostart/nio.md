![](/assets/netty-niostart-nio-服务端通信序列图.jpg)# Nio
`NIO`指非阻塞式`I/O`.

`JDK`的`NIO`为`Socket`和`ServerSocket` 提供了 `SocketChannel`和`ServerSocketChannel`两种不同的套接字通道实现。支持阻塞和非阻塞两种操作。

# NIO 类库简介
## 缓冲区Buffer
- NIO 库中，所有的操作都是用缓冲区处理的(读写)
- 缓冲区实质上是一个数组。通常是`ByteBuffer`,`JDK`基本数据类型(除Boolean)都有对应的缓冲区实现。 
- 提供对数据结构化访问以及维护读写位置(limit)等信息

![](/assets/image/netty-niostart-nio-buffer.jpg)

除了`ByteBuffer`外，其他的实例都有完全一样的操作。`ByteBuffer`方便网络的读写

## 通道 Channel
`Channel` 像自来水管一样，网络数据通过`Channel`读取和写入
- 通道是双向的（流是单向的且必须是InputStream和OutputStream的子类），读写可同时进行
- 全双工的；能更好的映射系统API,如UNIX网络模型中，底层操作系统的通道都是全双工的。同时支持读写操作。

![](/assets/netty-niostart-nio-channel.jpg)

前三层几乎上都是接口定义`Channel`功能。可分为两大类：
- SelectableChannel ： 网络读写
- FileChannel : 文件读写

## 多路复用器 Selector
多路复用是NIO的基础，非常重要！

- 提供选择已经就绪的任务的能力
   
   简单来说：`Selector`会不断轮训注册在其上的`Channel`，如果某个`Channel`上面发送读或则写事件，这个`Channel`就处于就绪状态，会被`Selector`轮询出来，然后通过`SelectorKey`可以获取就绪`Channel`的集合，进行后续的I/O操作
   
- 一个多路复用`Selector`可同时轮训多个`Channel`.JDK使用`eplll()`实现`select`，所以可以用一个线程负责`Selector`的轮训，就可以接入成千上万的客户端。   

## 服务端通信序列图
![](/assets/netty-niostart-nio-服务端通信序列图.jpg)

1. 打开sock通道
2. 绑定监听端口，并设置链接为非阻塞模式
3. 创建多路复用器，并启动线程
4. 将通道注册到 线程的多路复用器上，并监听`accept`事件
5. 多路复用器在线程run方法中无线轮询准备就绪的`Key`
6. 多路复用器监听到有新的客户端接入，处理新的接入请求，完成TCP三次握手，建立物理链路
7. 设置客户端链路为非阻塞模式
8. 将新接入的客户端链接注册到 多路复用器上，监听 读 操作。读取客户端的信息
9. 异步读取客户端请求信息到缓冲区
10. 对ByteBuff进行编码，如果有半包消息指针 reset。继续读取后续报文
11. 调用SocketChannel的异步write，将消息异步发送给客户端。

**注意：**
如果发送区TCP缓冲区满，会导致写半包，次数，需要注册监听写操作位，循环写，直到整包消息写入TCP缓冲区中。 这些内容Netty中有处理策略。

## 客户端通信序列图
![](/assets/netty-niostart-nio-客户端序列图.jpg)