# Nio
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