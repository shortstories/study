# Netty 기본 예제

## 테스트 환경

* Java 8
* Logback

## Discard 서버

* 8888 port를 listen하는 web server. 데이터를 받기만 할 뿐 아무런 동작도 하지 않음.
* port와 handler를 설정하는 main server
* client로부터 받은 입력 데이터를 처리하는 handler
* `telnet localhost 8888` 명령어로 테스트 가능

```java
public class DiscardServer {
  public static void main(String[] args) throws InterruptedException {
    EventLoopGroup bossGroup = new NioEventLoopGroup(1);
    EventLoopGroup workerGroup = new NioEventLoopGroup();

    try {
      ServerBootstrap bootstrap = new ServerBootstrap();
      bootstrap.group(bossGroup, workerGroup)
          .channel(NioServerSocketChannel.class)
          .childHandler(new ChannelInitializer<SocketChannel>() {
            @Override
            protected void initChannel(SocketChannel socketChannel) throws Exception {
              ChannelPipeline pipeline = socketChannel.pipeline();
              // 핸들러 설정
              pipeline.addLast(new DiscardServerHandler());
            }
          });

      // 포트 지정
      ChannelFuture future = bootstrap.bind(8888).sync();
      future.channel().closeFuture().sync();
    } finally {
      workerGroup.shutdownGracefully();
      bossGroup.shutdownGracefully();
    }
  }
}
```

```java
public class DiscardServerHandler extends SimpleChannelInboundHandler<Object> {
  private static final Logger logger = LoggerFactory.getLogger(DiscardServerHandler.class);

  @Override
  protected void channelRead0(ChannelHandlerContext ctx, Object obj) throws Exception {
    logger.debug("channelRead0");
  }

  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    logger.error("exceptionCaught", cause);
    ctx.close();
  }
}
```

## Echo 서버, 클라이언트

* 서버는 입력 받은 메세지를 그대로 다시 클라이언트에게 돌려줌
* SimpleChannelInboundHandler 대신에 ChannelInboundHandlerAdapter 사용
* 클라이언트는 채널이 활성화되면 "Hello, Netty!" 메세지를 서버에게 전송

### Server

```java
public class EchoServer {
  private static final Logger logger = LoggerFactory.getLogger(EchoServer.class);

  private EventLoopGroup bossGroup;
  private EventLoopGroup workerGroup;
  private ServerBootstrap bootstrap;

  public EchoServer() {
    bossGroup = new NioEventLoopGroup(1);
    workerGroup = new NioEventLoopGroup();

    bootstrap = new ServerBootstrap();
    bootstrap.group(bossGroup, workerGroup)
        .channel(NioServerSocketChannel.class)
        .childHandler(new ChannelInitializer<SocketChannel>() {
          @Override
          protected void initChannel(SocketChannel socketChannel) throws Exception {
            ChannelPipeline pipeline = socketChannel.pipeline();
            // 핸들러 설정
            pipeline.addLast(new EchoServerHandler());
          }
        });
  }

  public void start(final int port) throws InterruptedException {
    logger.info("start server...");
    ChannelFuture future = bootstrap.bind(port).sync();
    future.channel().closeFuture().sync();
  }

  public void close() {
    logger.info("close server...");
    workerGroup.shutdownGracefully();
    bossGroup.shutdownGracefully();
  }
}
```

```java
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
  private static final Logger logger = LoggerFactory.getLogger(EchoServerHandler.class);

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    logger.debug("receive message \"{}\"", ((ByteBuf)msg).toString(Charset.defaultCharset()));
    ctx.write(msg); // 버퍼에 쓰기
  }

  @Override
  public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
    ctx.flush(); // 채널 파이프라인에 저장된 버퍼 전송
  }

  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    logger.error("exceptionCaught", cause);
    ctx.close();
  }
}
```

### Client

```java
public class EchoClient {
  private static final Logger logger = LoggerFactory.getLogger(EchoClient.class);

  private EventLoopGroup group;
  private Bootstrap bootstrap;

  public EchoClient() {
    group = new NioEventLoopGroup();

    bootstrap = new Bootstrap();
    bootstrap.group(group)
        .channel(NioSocketChannel.class)
        .handler(new ChannelInitializer<SocketChannel>() {
          protected void initChannel(SocketChannel socketChannel) throws Exception {
            ChannelPipeline pipeline = socketChannel.pipeline();
            pipeline.addLast(new EchoClientHandler());
          }
        });
  }

  public void connect(final String host, final int port) throws InterruptedException {
    logger.info("connect to {}:{}", host, port);
    ChannelFuture future = bootstrap.connect(host, port).sync();
    future.channel().closeFuture().sync();
  }

  public void close() {
    logger.info("close client...");
    if (group != null) {
      group.shutdownGracefully();
    }
  }
}
```

```java
public class EchoClientHandler extends ChannelInboundHandlerAdapter {
  private static final Logger logger = LoggerFactory.getLogger(EchoClientHandler.class);

  @Override
  public void channelActive(ChannelHandlerContext ctx) throws Exception {
    String sendMessage = "Hello, Netty!";

    ByteBuf messageBuffer = Unpooled.buffer();
    messageBuffer.writeBytes(sendMessage.getBytes());

    ctx.writeAndFlush(messageBuffer);

    if (logger.isDebugEnabled()) {
      logger.debug("send message \"{}\"", sendMessage);
    }
  }

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    if (logger.isDebugEnabled()) {
      logger.debug("receive message \"{}\"", ((ByteBuf) msg).toString(Charset.defaultCharset()));
    }
  }

  @Override
  public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
    ctx.close();
  }

  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    logger.error("exceptionCaught", cause);
    ctx.close();
  }
}
```

