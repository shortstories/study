# Netty
## Discard 서버
- 8888 port를 listen하는 web server. 데이터를 받기만 할 뿐 아무런 동작도 하지 않음.
- port와 handler를 설정하는 main server
- client로부터 받은 입력 데이터를 처리하는 handler
- `telnet localhost 8888` 명령어로 테스트 가능

``` java
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

``` java
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

## Echo 서버
- Discard 서버와 대부분 동일. 입력받은 메세지를 그대로 돌려줌
- SimpleChannelInboundHandler 대신에 ChannelInboundHandlerAdapter 사용

``` java
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