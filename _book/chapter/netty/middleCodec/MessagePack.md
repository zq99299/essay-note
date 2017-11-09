# MessagePack 编解码
`MessagePack`是一个高效的二进制序列化框架。它像`JSON`支持多种语言，但它性能更快，序列化之后的码流也更小

本章主要内容包括：
* MessagePack 介绍
* MessagePack 编码器和解码器开发
* 粘包/半包支持

# MessagePack介绍

## 特点
* 编解码高效，性能高
* 序列化之后的码流小

## API介绍
```groovy
// https://mvnrepository.com/artifact/org.msgpack/msgpack
    compile group: 'org.msgpack', name: 'msgpack', version: '0.6.12'
```
```java
         // Create serialize objects.
        List<String> src = new ArrayList<String>();
        src.add("msgpack");
        src.add("kumofs");
        src.add("viver");


        MessagePack msgpack = new MessagePack();
        // Serialize
        byte[] raw = msgpack.write(src);
        System.out.println(raw.length);
        
        // 反序列化直接使用模板
        List<String> dst1 = msgpack.read(raw, Templates.tList(Templates.TString));
        System.out.println(dst1.get(0));
        System.out.println(dst1.get(1));
        System.out.println(dst1.get(2));

        // Or, Deserialze to Value then convert type.
        Value dynamic = msgpack.read(raw);
        List<String> dst2 = new Converter(dynamic)
                .read(Templates.tList(Templates.TString));
        System.out.println(dst2.get(0));
        System.out.println(dst2.get(1));
        System.out.println(dst2.get(2));
```

# 编解码开发
## 编码器
```java
public class MsgpackEncoder extends MessageToByteEncoder {
    @Override
    protected void encode(ChannelHandlerContext ctx, Object msg, ByteBuf out) throws Exception {
        try {
            MessagePack messagePack = new MessagePack();
            byte[] raw = messagePack.write(msg);
            out.writeBytes(raw);
        } catch (Exception e) {
            // 这里的异常抛出去就被框架吃掉了。看不到异常的
            e.printStackTrace();
        }
    }
}
```
## 解码器
```java
public class MsgpackDecoder extends MessageToMessageDecoder<ByteBuf> {

    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf msg, List<Object> out) throws Exception {

        // 获取解码数组：ByteBuf实在没有看懂怎么使用的
        int length = msg.readableBytes();
        byte[] bodys = new byte[length];
        msg.getBytes(msg.readerIndex(), bodys, 0, length);

        MessagePack messagePack = new MessagePack();
        Value read = messagePack.read(bodys);
        out.add(read);
    }
}
```
## 服务端和客户端
```java
public class EchoServer {
    Logger log = LoggerFactory.getLogger(getClass());

    public static void main(String[] args) throws InterruptedException {
        new EchoServer().bind(8086);
    }

    public void bind(int port) throws InterruptedException {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup();
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap starp = new ServerBootstrap();
            starp.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 1024)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast("msgpack decoder", new MsgpackDecoder());
                            ch.pipeline().addLast("msgpack encoder", new MsgpackEncoder());
                            ch.pipeline().addLast(new EchoServerHandler());
                        }
                    });
            ChannelFuture channelFuture = starp.bind(port).sync();
            channelFuture.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public class EchoServerHandler extends ChannelHandlerAdapter {
        @Override
        public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
            // 这里得到的ms 是一个 ArrayValueImpl实例。暂时不能还原成对象
            log.info("===== : " + msg);
        }

        @Override
        public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
            cause.printStackTrace();
            ctx.close();
        }
    }
}

public class EchoClient {
    Logger log = LoggerFactory.getLogger(getClass());

    public static void main(String[] args) throws InterruptedException {
        new EchoClient().connect("127.0.0.1", 8086);
    }

    public void connect(String host, int port) throws InterruptedException {
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .option(ChannelOption.TCP_NODELAY, true)
                    .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 3000)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast("msgpack decoder", new MsgpackDecoder());
                            ch.pipeline().addLast("msgpack encoder", new MsgpackEncoder());
                            ch.pipeline().addLast(new EchoClientHandler());
                        }
                    });
            // 发起异步链接操作
            ChannelFuture future = bootstrap.connect(host, port).sync();
            // 同步阻塞，链路关闭才被唤醒
            future.channel().closeFuture().sync();

        } finally {
            //优雅退出，释放NIO线程组
            group.shutdownGracefully();
        }
    }

    public class EchoClientHandler extends ChannelHandlerAdapter {
        @Override
        public void channelActive(ChannelHandlerContext ctx) throws Exception {
            for (int i = 0; i < 100; i++) {
                ctx.write(new UserInfo(i, "name" + i));  // 不发一条刷新一条的话，有粘包/拆包问题，服务端收到的消息是不完整的
//                ctx.writeAndFlush(new UserInfo(i, "name" + i));
            }
            ctx.flush();
        }

        @Override
        public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
            log.info("===== : " + msg);
        }

        @Override
        public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
            ctx.flush();
        }

        @Override
        public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
            cause.printStackTrace();
            ctx.close();
        }
    }
}
```

JavaBean

```java
// 必须加此注解否则异常：Cannot find template for class messagepack.UserInfo class.  Try to add @Message annotation to the class or call MessagePack.register(Type).
@Message
public class UserInfo {
    private int age;
    private String name;

    /**
     * 要注意留一个空参构造，否则会反射失败
     * 二月 28, 2017 11:49:55 上午 org.msgpack.template.builder.BuildContext build
         严重: builder:
         {
         if (!$3 && $1.trySkipNil()) {
         return null;
         }
         messagepack.UserInfo _(双$)_t;
         if ($2 == null) {
         _(双$)_t = new messagepack.UserInfo();
         } else {
         _$$_t = (messagepack.UserInfo) $2;
         }
     */
    public UserInfo() {
    }

    public UserInfo(int age, String name) {
        this.age = age;
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

## 粘包/拆包解决
这里使用常用的，在消息头中增加报文长度策略：使用netty自带的`LengthFieldBasedFrameDecoder`解码器和`LengthFieldPrepender`编码处理器；

客户端和服务端的配置都是一样的。都需要增加
```java
ChannelPipeline pipeline = ch.pipeline();
// 这里使用在请求里增加 消息长度解决方案

// 在自定义解码之前增加处理半包的解码器
pipeline.addLast("frame decoder", new LengthFieldBasedFrameDecoder(65535,0,2,0,2));
// 这里接收到的就永远是整包消息了
pipeline.addLast("msgpack decoder", new MsgpackDecoder());
// 编码之前增加 两个字节的消息长度，
pipeline.addLast("frame encoder",new LengthFieldPrepender(2));
pipeline.addLast("msgpack encoder", new MsgpackEncoder());
pipeline.addLast(new EchoServerHandler());

```