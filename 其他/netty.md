```java
public class NioServer {

    public static void main(String[] args) throws InterruptedException {
        EventLoopGroup bossGroup=new NioEventLoopGroup(1);
        EventLoopGroup workGroup=new NioEventLoopGroup();
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap.group(bossGroup,workGroup)
                .channel(NioServerSocketChannel.class)
                .childOption(ChannelOption.TCP_NODELAY,true)
                .childAttr(AttributeKey.newInstance("childattr"),"childvalue")
                .handler(new MyHandler())
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        //socketChannel.pipeline().addLast()
                    }
                });
        ChannelFuture future=serverBootstrap.bind(9000).sync();
        future.channel().closeFuture();

        bossGroup.shutdownGracefully();
        workGroup.shutdownGracefully();
        /**
         * 反射创建服务端new channel
         * .channel  newSocket()通过jdk来创建地城jdk channek
         * nioserversocketchannelconfig   tcp参数配置
         * abstractniochannel()
         * configureblocking(false)
         * abstracchannel id unsafe pipeline
         */

        /**
         * 初始化服务端init channel
         * bind
         * initandregister
         * newchannel
         * init
         */
        /**
         * chanel  注册到register selector
         * abstractchannel @ register()
         * this.eventloop=eventloop
         * register0  实际注册
         * doregister  jdk底层注册
         * invokehandleraddedifneeded
         * firechannelregistered()
         */
        /**
         * 端口绑定 do bind
         * abstracunsafe.bind
         * dobind
         * javachannel.bind   jdk底层绑定
         * pipeline。firechannelactive 传播
         *headcontext.readifisautoread
         */
    }

    static class MyHandler extends ChannelInboundHandlerAdapter{
        public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
            System.out.println("register");
        }

        public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
        }

        public void channelActive(ChannelHandlerContext ctx) throws Exception {
            System.out.println("active");
        }
    }
}

```

默认 netty服务端起多少  



线程何时启动？  线程组默认 cpu*2

MultithreadEventExecutorGroup @    new threadpertaskexecutor 每次执行任务都会创建一个线程实体  

for new Child 构造nioeventloop    保存线程执行器threadpertaskexecutor  创建mpscqueue接受外部线程去执行    创建一个selector      

choosefactory.choose           nioeventloop[] 绑定



nioeventloop run()

run->while(true)



select 检查是否有io事件

​	deadline以及任务穿插逻辑处理

​	阻塞式select

​	避免jdk空轮询bug    次数多了就替换



processselectedkeys  处理io事件

​	openselector 事件轮询器

​	selected     keyset优化   实际使用数组实现 替换selectedkeyset

​	processelectedkeyoptimized

runalltasks    处理异步任务队列



netty如何保证异步串行无锁化？