# AIO/NIO2.0
- Nio : 是JDK1.4引入的,使用起来编写太复杂了
- Aio : 是JDK1.7 引入的？，是真正的异步非阻塞I/O
- NIO2.0 ： 不需要通过多路复用器(Selector)轮训来实现异步读写

异步通道（文件和套接字）通过以下两种方式获取操作结果：
- 通过`java.util.concurrent.Future`来标识异步操作结果
- 在执行异步操作的时候传入一个`java.nio.channels`

## 服务端
```java
/**
 * @author zhuqiang
 * @version 1.0.1 2017/2/16 13:17
 * @date 2017/2/16 13:17
 * @since 1.0
 */
public class TimeServer {
    public static void main(String[] args) {
        int port = 8086;
        new Thread(new TimeServerHandler(port)).start();
    }
}

class TimeServerHandler implements Runnable {
    private int port;
    private CountDownLatch latch;
    AsynchronousServerSocketChannel ass;

    public TimeServerHandler(int port) {
        this.port = port;
        try {
            ass = AsynchronousServerSocketChannel.open();
            ass.bind(new InetSocketAddress(port));
            System.out.println("=== 服务器初始化成功.port = " + port);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void run() {
        latch = new CountDownLatch(1);
        doAccept(); //处理客户端链接
        try {
            latch.await(); // 不使用死循环的方式来阻止线程结束，使用发令枪的方法更优雅
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private void doAccept() {
        ass.accept(this, new CompletionHandler<AsynchronousSocketChannel, TimeServerHandler>() {
            @Override
            public void completed(AsynchronousSocketChannel result, TimeServerHandler attachment) {
                // 接受下一个链接
                attachment.ass.accept(attachment, this);
                // 获得了该链接，就可以处理该链接
                ByteBuffer buffer = ByteBuffer.allocate(1024);
                // 读取数据到buffer中、业务对象传递给回调处理中、通知回调
                result.read(buffer, buffer, new ReadCompletionHandler(result));
            }

            @Override
            public void failed(Throwable exc, TimeServerHandler attachment) {
                exc.printStackTrace();
                // 链接失败 发令枪发令，结束当前线程
                attachment.latch.countDown();
            }
        });
    }
}

class ReadCompletionHandler implements CompletionHandler<Integer, ByteBuffer> {
    private AsynchronousSocketChannel channel;

    public ReadCompletionHandler(AsynchronousSocketChannel channel) {
        this.channel = channel;
    }

    @Override
    public void completed(Integer result, ByteBuffer attachment) {
        attachment.flip();
        byte[] body = new byte[attachment.remaining()];
        attachment.get(body);
        String req = new String(body, Charset.forName("UTF-8"));
        System.out.println("=== 收到消息：" + req);
        // 响应消息
        doWrite("当前时间:" + new Date());
    }

    private void doWrite(String res) {
        byte[] bytes = res.getBytes();
        ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);
        writeBuffer.put(bytes);
        writeBuffer.flip();
        channel.write(writeBuffer, writeBuffer, new CompletionHandler<Integer, ByteBuffer>() {
            @Override
            public void completed(Integer result, ByteBuffer attachment) {
                // 如果没有发送完成，继续发送
                if (attachment.hasRemaining()) {
                    channel.write(attachment, attachment, this);
                }
            }

            @Override
            public void failed(Throwable exc, ByteBuffer attachment) {
                try {
                    channel.close();
                    exc.printStackTrace();
                    System.out.println("== 发送失败退出");
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {
        try {
            channel.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```
## 客户端
```java
/**
 * @author zhuqiang
 * @version 1.0.1 2017/2/16 14:07
 * @date 2017/2/16 14:07
 * @since 1.0
 */
public class TimeClient {
    public static void main(String[] args) {
        int port = 8086;
        new Thread(new TimeClientHandler("127.0.0.1", port), "Aio-001").start();
    }
}

class TimeClientHandler implements Runnable, CompletionHandler<Void, TimeClientHandler> {
    private Logger log = LoggerFactory.getLogger(getClass());
    private AsynchronousSocketChannel asc;
    private String host;
    private int port;
    private CountDownLatch latch;

    public TimeClientHandler(String host, int port) {
        this.host = host;
        this.port = port;
        try {
            asc = AsynchronousSocketChannel.open();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void run() {
        latch = new CountDownLatch(1);
        asc.connect(new InetSocketAddress(host, port), this, this);
        try {
            latch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        try {
            asc.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void completed(Void result, TimeClientHandler attachment) {
        try {
            // 发送，也可以写容错，判断是否发送完成，半包的问题。这里就不处理半包问题了
            attachment.asc.write(ByteBuffer.wrap("你好呀".getBytes("UTF-8")));
            ByteBuffer readBuffer = ByteBuffer.allocate(1024);
            // 异步读取操作，所以还是需要回调函数 处理读到的数据
            asc.read(readBuffer, readBuffer, new CompletionHandler<Integer, ByteBuffer>() {
                @Override
                public void completed(Integer result, ByteBuffer attachment) {
                    attachment.flip();
                    byte[] body = new byte[attachment.remaining()];
                    attachment.get(body);
                    try {
                        log.info("== 收到消息：" + new String(body, "UTF-8"));
                    } catch (UnsupportedEncodingException e) {
                        e.printStackTrace();
                    }
                    latch.countDown();
                }

                @Override
                public void failed(Throwable exc, ByteBuffer attachment) {
                    exc.printStackTrace();
                }
            });

        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void failed(Throwable exc, TimeClientHandler attachment) {
        try {
            asc.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```
运行后，能正常通信，但是还是不能用web浏览器直接访问。不能接收消息，但是服务端能接收到请求

# 总结
从打印的日志来看，回调函数是通过线程回调的。所以说，新的异步大量基于回调函数，而每个客户端的链接通道是通过jdk底层的线程框架来运行的。
