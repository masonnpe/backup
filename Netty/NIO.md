---
title: Nio
categories: Netty
tags: [Nio]
---

**初识NIO：**

​    在 JDK 1. 4 中 新 加入 了 NIO( New Input/ Output) 类, 引入了一种基于通道和缓冲区的 I/O 方式，它可以使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆的 DirectByteBuffer 对象作为这块内存的引用进行操作，避免了在 Java 堆和 Native 堆中来回复制数据。

​    NIO 是一种同步非阻塞的 IO 模型。同步是指线程不断轮询 IO 事件是否就绪，非阻塞是指线程在等待 IO 的时候，可以同时做其他任务。同步的核心就是 Selector，Selector 代替了线程本身轮询 IO 事件，避免了阻塞同时减少了不必要的线程消耗；非阻塞的核心就是通道和缓冲区，当 IO 事件就绪时，可以通过写道缓冲区，保证 IO 的成功，而无需线程阻塞式地等待。

**Buffer：**

​    为什么说NIO是基于缓冲区的IO方式呢？因为，当一个链接建立完成后，IO的数据未必会马上到达，为了当数据到达时能够正确完成IO操作，在BIO（阻塞IO）中，等待IO的线程必须被阻塞，以全天候地执行IO操作。为了解决这种IO方式低效的问题，引入了缓冲区的概念，当数据到达时，可以预先被写入缓冲区，再由缓冲区交给线程，因此线程无需阻塞地等待IO。

**通道：**

​    当执行：SocketChannel.write(Buffer)，便将一个 buffer 写到了一个通道中。如果说缓冲区还好理解，通道相对来说就更加抽象。网上博客难免有写不严谨的地方，容易使初学者感到难以理解。

​    引用 Java NIO 中权威的说法：通道是 I/O 传输发生时通过的入口，而缓冲区是这些数 据传输的来源或目标。对于离开缓冲区的传输，您想传递出去的数据被置于一个缓冲区，被传送到通道。对于传回缓冲区的传输，一个通道将数据放置在您所提供的缓冲区中。

​    例如 有一个服务器通道 ServerSocketChannel serverChannel，一个客户端通道 SocketChannel clientChannel；服务器缓冲区：serverBuffer，客户端缓冲区：clientBuffer。

​    当服务器想向客户端发送数据时，需要调用：clientChannel.write(serverBuffer)。当客户端要读时，调用 clientChannel.read(clientBuffer)

​    当客户端想向服务器发送数据时，需要调用：serverChannel.write(clientBuffer)。当服务器要读时，调用 serverChannel.read(serverBuffer)

​    这样，通道和缓冲区的关系似乎更好理解了。在实践中，未必会出现这种双向连接的蠢事（然而这确实存在的，后面的内容还会涉及），但是可以理解为在NIO中：如果想将Data发到目标端，则需要将存储该Data的Buffer，写入到目标端的Channel中，然后再从Channel中读取数据到目标端的Buffer中。

**Selector：**

​    通道和缓冲区的机制，使得线程无需阻塞地等待IO事件的就绪，但是总是要有人来监管这些IO事件。这个工作就交给了selector来完成，这就是所谓的同步。

​    Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。

​    要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪，这就是所说的轮询。一旦这个方法返回，线程就可以处理这些事件。

​    Selector中注册的感兴趣事件有：

- OP_ACCEPT
- OP_CONNECT 
- OP_READ 
- OP_WRITE

**优化：**

​    一种优化方式是：将Selector进一步分解为Reactor，将不同的感兴趣事件分开，每一个Reactor只负责一种感兴趣的事件。这样做的好处是：1、分离阻塞级别，减少了轮询的时间；2、线程无需遍历set以找到自己感兴趣的事件，因为得到的set中仅包含自己感兴趣的事件。

![img](http://imglf1.ph.126.net/9AtBKwQ8vHsko1RPRb0sew==/6631645009304717344.jpg)

**NIO和epoll：**

​    epoll是Linux内核的IO模型。我想一定有人想问，AIO听起来比NIO更加高大上，为什么不使用AIO？AIO其实也有应用，但是有一个问题就是，Linux是不支持AIO的，因此基于AIO的程序运行在Linux上的效率相比NIO反而更低。而Linux是最主要的服务器OS，因此相比AIO，目前NIO的应用更加广泛。

​    说到这里，可能你已经明白了，epoll一定和NIO有着很深的因缘。没错，如果仔细研究epoll的技术内幕，你会发现它确实和NIO非常相似，都是基于“通道”和缓冲区的，也有selector，只是在epoll中，通道实际上是操作系统的“管道”。和NIO不同的是，NIO中，解放了线程，但是需要由selector阻塞式地轮询IO事件的就绪；而epoll中，IO事件就绪后，会自动发送消息，通知selector：“我已经就绪了。”可以认为，Linux的epoll是一种效率更高的NIO。

**NIO轶事：**

​    一篇有意思的[博客](http://blog.csdn.net/haoel/article/details/2224069)，讲的 Java selector.open() 的时候，会创建一个自己和自己的链接（windows上是tcp，linux上是通道）

​    这么做的原因：可以从 Apache Mina 中窥探。在 Mina 中，有如下机制：

1. Mina框架会创建一个Work对象的线程。
2. Work对象的线程的run()方法会从一个队列中拿出一堆Channel，然后使用Selector.select()方法来侦听是否有数据可以读/写。
3. 最关键的是，在select的时候，如果队列有新的Channel加入，那么，Selector.select()会被唤醒，然后重新select最新的Channel集合。
4. 要唤醒select方法，只需要调用Selector的wakeup()方法。

​    而一个阻塞在select上的线程有以下三种方式可以被唤醒：

1. 有数据可读/写，或出现异常。
2. 阻塞时间到，即time out。
3. 收到一个non-block的信号。可由kill或pthread_kill发出。

​    首先 2 可以排除，而第三种方式，只在linux中存在。因此，Java NIO为什么要创建一个自己和自己的链接：就是如果想要唤醒select，只需要朝着自己的这个loopback连接发点数据过去，于是，就可以唤醒阻塞在select上的线程了。



[《Java NIO编写Socket服务器的一个例子》](https://blog.csdn.net/xidianliuy/article/details/51612676)

























1.Java NIO概览
首先，熟悉一下NIO的主要组成部分：
Bufer，高效的数据容器，除了布尔类型，所有原始数据类型都有相应的Bufer实现。
Channel，类似在Linux之类操作系统上看到的文件描述符，是NIO中被用来支持批量式IO操作的一种抽象。
File或者Socket，通常被认为是比较高层次的抽象，而Channel则是更加操作系统底层的一种抽象，这也使得NIO得以充分利用现代操作系统底层机制，获得特定场景的性能优
化，例如，DMA（Direct Memory Access）等。不同层次的抽象是相互关联的，我们可以通过Socket获取Channel，反之亦然。
Selector，是NIO实现多路复用的基础，它提供了一种高效的机制，可以检测到注册在Selector上的多个Channel中，是否有Channel处于就绪状态，进而实现了单线程对
多Channel的高效管理。
Selector同样是基于底层操作系统机制，不同模式、不同版本都存在区别，例如，在最新的代码库里，相关实现如下：
Linux上依赖于epoll（http://hg.openjdk.java.net/jdk/jdk/fle/d8327f838b88/src/java.base/linux/classes/sun/nio/ch/EPollSelectorImpl.java）。
Windows上NIO2（AIO）模式则是依赖于iocp（http://hg.openjdk.java.net/jdk/jdk/fle/d8327f838b88/src/java.base/windows/classes/sun/nio/ch/Iocp.java）。
Chartset，提供Unicode字符串定义，NIO也提供了相应的编解码器等，例如，通过下面的方式进行字符串到ByteBufer的转换：
Charset.defaultCharset().encode("Hello world!"));
2.NIO能解决什么问题？
下面我通过一个典型场景，来分析为什么需要NIO，为什么需要多路复用。设想，我们需要实现一个服务器应用，只简单要求能够同时服务多个客户端请求即可。
使用java.io和java.net中的同步、阻塞式API，可以简单实现。
public class DemoServer extends Thread {
private ServerSocket serverSocket;
public int getPort() {
return serverSocket.getLocalPort();
}
public void run() {
try {
极客时间
serverSocket = new ServerSocket(0);
while (true) {
Socket socket = serverSocket.accept();
RequesHandler requesHandler = new RequesHandler(socket);
requesHandler.sart();
}
} catch (IOException e) {
e.printStackTrace();
} fnally {
if (serverSocket != null) {
try {
serverSocket.close();
} catch (IOException e) {
e.printStackTrace();
}
;
}
}
}
public satic void main(String[] args) throws IOException {
DemoServer server = new DemoServer();
server.sart();
try (Socket client = new Socket(InetAddress.getLocalHos(), server.getPort())) {
BuferedReader buferedReader = new BuferedReader(new InputStreamReader(client.getInputStream()));
buferedReader.lines().forEach(s -> Sysem.out.println(s));
}
}
}
// 简化实现，不做读取，直接发送字符串
class RequesHandler extends Thread {
private Socket socket;
RequesHandler(Socket socket) {
this.socket = socket;
}
@Override
public void run() {
try (PrintWriter out = new PrintWriter(socket.getOutputStream());) {
out.println("Hello world!");
out.fush();
} catch (Exception e) {
e.printStackTrace();
}
}
}
其实现要点是：
服务器端启动ServerSocket，端口0表示自动绑定一个空闲端口。
调用accept方法，阻塞等待客户端连接。
利用Socket模拟了一个简单的客户端，只进行连接、读取、打印。
当连接建立后，启动一个单独线程负责回复客户端请求。
这样，一个简单的Socket服务器就被实现出来了。
思考一下，这个解决方案在扩展性方面，可能存在什么潜在问题呢？
大家知道Java语言目前的线程实现是比较重量级的，启动或者销毁一个线程是有明显开销的，每个线程都有单独的线程栈等结构，需要占用非常明显的内存，所以，每一个Client启
动一个线程似乎都有些浪费。
那么，稍微修正一下这个问题，我们引入线程池机制来避免浪费。
serverSocket = new ServerSocket(0);
executor = Executors.newFixedThreadPool(8);
while (true) {
Socket socket = serverSocket.accept();
RequesHandler requesHandler = new RequesHandler(socket);
executor.execute(requesHandler);
}
这样做似乎好了很多，通过一个固定大小的线程池，来负责管理工作线程，避免频繁创建、销毁线程的开销，这是我们构建并发服务的典型方式。这种工作方式，可以参考下图来理
解。
极客时间
如果连接数并不是非常多，只有最多几百个连接的普通应用，这种模式往往可以工作的很好。但是，如果连接数量急剧上升，这种实现方式就无法很好地工作了，因为线程上下文切
换开销会在高并发时变得很明显，这是同步阻塞方式的低扩展性劣势。
NIO引入的多路复用机制，提供了另外一种思路，请参考我下面提供的新的版本。
public class NIOServer extends Thread {
public void run() {
try (Selector selector = Selector.open();
ServerSocketChannel serverSocket = ServerSocketChannel.open();) {// 创建Selector和Channel
serverSocket.bind(new InetSocketAddress(InetAddress.getLocalHos(), 8888));
serverSocket.confgureBlocking(false);
// 注册到Selector，并说明关注点
serverSocket.regiser(selector, SelectionKey.OP_ACCEPT);
while (true) {
selector.select();// 阻塞等待就绪的Channel，这是关键点之一
Set selectedKeys = selector.selectedKeys();
Iterator iter = selectedKeys.iterator();
while (iter.hasNext()) {
SelectionKey key = iter.next();
// 生产系统中一般会额外进行就绪状态检查
sayHelloWorld((ServerSocketChannel) key.channel());
iter.remove();
}
}
} catch (IOException e) {
e.printStackTrace();
}
}
private void sayHelloWorld(ServerSocketChannel server) throws IOException {
try (SocketChannel client = server.accept();) { client.write(Charset.defaultCharset().encode("Hello world!"));
}
}
// 省略了与前面类似的main
}
这个非常精简的样例掀开了NIO多路复用的面纱，我们可以分析下主要步骤和元素：
首先，通过Selector.open()创建一个Selector，作为类似调度员的角色。
然后，创建一个ServerSocketChannel，并且向Selector注册，通过指定SelectionKey.OP_ACCEPT，告诉调度员，它关注的是新的连接请求。
注意，为什么我们要明确配置非阻塞模式呢？这是因为阻塞模式下，注册操作是不允许的，会抛出IllegalBlockingModeException异常。
Selector阻塞在select操作，当有Channel发生接入请求，就会被唤醒。
在sayHelloWorld方法中，通过SocketChannel和Bufer进行数据操作，在本例中是发送了一段字符串。
可以看到，在前面两个样例中，IO都是同步阻塞模式，所以需要多线程以实现多任务处理。而NIO则是利用了单线程轮询事件的机制，通过高效地定位就绪的Channel，来决定做什
么，仅仅select阶段是阻塞的，可以有效避免大量客户端连接时，频繁线程切换带来的问题，应用的扩展能力有了非常大的提高。下面这张图对这种实现思路进行了形象地说明。
极客时间
在Java 7引入的NIO 2中，又增添了一种额外的异步IO模式，利用事件和回调，处理Accept、Read等操作。 AIO实现看起来是类似这样子：
AsynchronousServerSocketChannel serverSock = AsynchronousServerSocketChannel.open().bind(sockAddr);
serverSock.accept(serverSock, new CompletionHandler<>() { //为异步操作指定CompletionHandler回调函数
@Override
public void completed(AsynchronousSocketChannel sockChannel, AsynchronousServerSocketChannel serverSock) {
serverSock.accept(serverSock, this);
// 另外一个 write（sock，CompletionHandler{}）
sayHelloWorld(sockChannel, Charset.defaultCharset().encode
("Hello World!"));
}
// 省略其他路径处理方法...
});
鉴于其编程要素（如Future、CompletionHandler等），我们还没有进行准备工作，为避免理解困难，我会在专栏后面相关概念补充后的再进行介绍，尤其
是Reactor、Proactor模式等方面将在Netty主题一起分析，这里我先进行概念性的对比：
基本抽象很相似，AsynchronousServerSocketChannel对应于上面例子中的ServerSocketChannel；AsynchronousSocketChannel则对应SocketChannel。
业务逻辑的关键在于，通过指定CompletionHandler回调接口，在accept/read/write等关键节点，通过事件机制调用，这是非常不同的一种编程思路。
今天我初步对Java提供的IO机制进行了介绍，概要地分析了传统同步IO和NIO的主要组成，并根据典型场景，通过不同的IO模式进行了实现与拆解。专栏下一讲，我还将继续分
析Java IO的主题。