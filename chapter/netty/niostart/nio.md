# Nio
`NIO`指非阻塞式`I/O`.

`JDK`的`NIO`为`Socket`和`ServerSocket` 提供了 `SocketChannel`和`ServerSocketChannel`两种不同的套接字通道实现。支持阻塞和非阻塞两种操作。

# NIO 类库简介
## 缓冲区Buffer
- NIO 库中，所有的操作都是用缓冲区处理的(读写)
- 缓冲区实质上是一个数组。通常是`ByteBuffer`,`JDK`基本数据类型(除Boolean)都有对应的缓冲区实现。 
- 提供对数据结构化访问以及维护读写位置(limit)等信息