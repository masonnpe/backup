---
title: Java IO
categories: Java基础
tags: [IO,NIO]
---

<!--more-->







Java有多种比较典型的文件拷贝实现方式，比如：

```java
private static void copy(String sourcePath,String desPath){
    try {
        FileChannel sourceChannel = new FileInputStream(new File(sourcePath)).getChannel();
        FileChannel desChannel = new FileOutputStream(new File(desPath)).getChannel();
        for (long count = sourceChannel.size();count>0;){
            long transferred=sourceChannel.transferTo(sourceChannel.position(),count,desChannel);
            try {
                sourceChannel.position(sourceChannel.position()+transferred);
            } catch (IOException e) {
                e.printStackTrace();
            }
            count-=transferred;
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

知识扩展
1.拷贝实现机制分析

而基于NIO transferTo的实现方式，在Linux和Unix上，则会使用到零拷贝技术省去了上下文切换的开销和不必要的内存拷贝，进而可能提高应用
拷贝性能。注意，transferTo不仅仅是可以用在文件拷贝中，与其类似的，例如读取磁盘文件，然后进行Socket发送，同样可以享受这种机制带来的性能和扩展性提高。




### 通道

Channel 通道是对原 I/O 包中的流的模拟，可以通过它读取和写入数据，所有数据都通过 `Buffer` 对象来处理。字节不会直接写入通道中，也不能直接从通道中读取字节。就像不能在高速公路正中间停下来一样，可以在加油站加油一样

通道与流的不同之处在于通道是双向的，而流只是在一个方向上移动，可以用于读、写或者同时用于读写

- FileChannel：从文件中读写数据
- DatagramChannel：通过 UDP 读写网络中数据
- SocketChannel：通过 TCP 读写网络中数据
- ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel

### 缓冲区

Buffer 缓冲区是一个容器，包含包含一些要写入或者刚读出的数据。发送给一个通道的所有对象都必须首先放到缓冲区中，从通道中读取的任何数据都要读到缓冲区中

#### 状态变量

每一个缓冲区都有复杂的内部统计机制，它会跟踪缓冲区已经读了多少数据以及还有多少空间可以容纳更多的数据，会跟踪缓冲区包含多少数据以及还有多少数据要写入。每一个读/写操作都会改变缓冲区的状态。通过记录和跟踪这些变化，缓冲区就可能够内部地管理自己的资源。状态变量就是内部统计机制的关键。可以用三个值指定缓冲区在任意时刻的状态

* position 表示当前已经读写的字节位置
* limit 表示还可以读写的字节数
* capacity 缓冲区中的最大数据容量

要将数据写到输出通道中。在这之前，必须调用 `flip()` 方法

1. 将 `limit` 设置为当前 `position`
2. 将 `position` 设置为 0
3. 写`position` 到`limit` 间的数据

`clear()`重设缓冲区以便接收更多的字节

1. 将 `limit` 设置为与 `capacity` 相同
2. 设置 `position` 为 0
3. 回到最初的模样

#### 访问方法

 `get()` 和 `put()` 方法直接访问缓冲区中的数据

---

### 选择器

NIO 实现了 IO 多路复用中的 Reactor 模型，一个线程 Thread 使用一个选择器 Selector 通过轮询的方式去监听多个通道 Channel 上的事件，从而让一个线程就可以处理多个事件。

通过配置监听的通道 Channel 为非阻塞，那么当 Channel 上的 IO 事件还未到达时，就不会进入阻塞状态一直等待，而是继续轮询其它 Channel，找到 IO 事件已经到达的 Channel 执行。

因为创建和切换线程的开销很大，因此使用一个线程来处理多个事件而不是一个线程处理一个事件，对于 IO 密集型的应用具有很好地性能。

应该注意的是，只有套接字 Channel 才能配置为非阻塞，而 FileChannel 不能，为 FileChannel 配置非阻塞也没有意义。

### 读文件Demo

```java
File file=new File("C:\\Users\\桔子\\Desktop\\test.txt");
		try {
			FileInputStream fis=new FileInputStream(file);
			// 获取管道
			FileChannel channel=fis.getChannel();
			// 创建缓冲区
			ByteBuffer buffer=ByteBuffer.allocate(1024);
			// 数据读到缓冲区
			channel.read(buffer);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
```

### 补充

[NIO入门](https://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html)

[Java NIO浅析](https://tech.meituan.com/nio.html)







## NIO与AIO学习总结

### [一 Java NIO 概览](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483956&idx=1&sn=57692bc5b7c2c6dfb812489baadc29c9&chksm=fd985455caefdd4331d828d8e89b22f19b304aa87d6da73c5d8c66fcef16e4c0b448b1a6f791#rd)



### [二 Java NIO 之 Buffer(缓冲区)](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483961&idx=1&sn=f67bef4c279e78043ff649b6b03fdcbc&chksm=fd985458caefdd4e3317ccbdb2d0a5a70a5024d3255eebf38183919ed9c25ade536017c0a6ba#rd)

1. **Buffer(缓冲区)介绍:**
   - Java NIO Buffers用于和NIO Channel交互。 我们从Channel中读取数据到buffers里，从Buffer把数据写入到Channels；
   - Buffer本质上就是一块内存区；
   - 一个Buffer有三个属性是必须掌握的，分别是：capacity容量、position位置、limit限制。

2. **Buffer的常见方法**
   - Buffer clear()
   - Buffer flip()
   - Buffer rewind()
   - Buffer position(int newPosition)


### [三 Java NIO 之 Channel（通道）](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483966&idx=1&sn=d5cf18c69f5f9ec2aff149270422731f&chksm=fd98545fcaefdd49296e2c78000ce5da277435b90ba3c03b92b7cf54c6ccc71d61d13efbce63#rd)

1. **Channel（通道）介绍**
   - 通常来说NIO中的所有IO都是从 Channel（通道） 开始的。 
   - NIO Channel通道和流的区别：
2. **FileChannel的使用**
3. **SocketChannel和ServerSocketChannel的使用**
4. **️DatagramChannel的使用**
5. **Scatter / Gather**
   - Scatter: 从一个Channel读取的信息分散到N个缓冲区中(Buufer).
   - Gather: 将N个Buffer里面内容按照顺序发送到一个Channel.
6. **通道之间的数据传输**
   - 在Java NIO中如果一个channel是FileChannel类型的，那么他可以直接把数据传输到另一个channel。
   - transferFrom() :transferFrom方法把数据从通道源传输到FileChannel
   - transferTo() :transferTo方法把FileChannel数据传输到另一个channel

### [四 Java NIO之Selector（选择器）](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483970&idx=1&sn=d5e2b133313b1d0f32872d54fbdf0aa7&chksm=fd985423caefdd354b587e57ce6cf5f5a7bec48b9ab7554f39a8d13af47660cae793956e0f46#rd)

1. **Selector（选择器）介绍**

   - Selector 一般称 为选择器 ，当然你也可以翻译为 多路复用器 。它是Java NIO核心组件中的一个，用于检查一个或多个NIO Channel（通道）的状态是否处于可读、可写。如此可以实现单线程管理多个channels,也就是可以管理多个网络链接。
   - 使用Selector的好处在于： 使用更少的线程来就可以来处理通道了， 相比使用多个线程，避免了线程上下文切换带来的开销。

2. **Selector（选择器）的使用方法介绍**

   - Selector的创建

   ```java
   Selector selector = Selector.open();
   ```

   - 注册Channel到Selector(Channel必须是非阻塞的)

   ```java
   channel.configureBlocking(false);
   SelectionKey key = channel.register(selector, Selectionkey.OP_READ);
   ```

   - SelectionKey介绍

     一个SelectionKey键表示了一个特定的通道对象和一个特定的选择器对象之间的注册关系。

   - 从Selector中选择channel(Selecting Channels via a Selector)

     选择器维护注册过的通道的集合，并且这种注册关系都被封装在SelectionKey当中.

   - 停止选择的方法

     wakeup()方法 和close()方法。

3. **模板代码**

   有了模板代码我们在编写程序时，大多数时间都是在模板代码中添加相应的业务代码。

4. **客户端与服务端简单交互实例**



### [五 Java NIO之拥抱Path和Files](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483976&idx=1&sn=2296c05fc1b840a64679e2ad7794c96d&chksm=fd985429caefdd3f48e2ee6fdd7b0f6fc419df90b3de46832b484d6d1ca4e74e7837689c8146&token=537240785&lang=zh_CN#rd)

**一 文件I/O基石：Path：**

- 创建一个Path
- File和Path之间的转换，File和URI之间的转换
- 获取Path的相关信息
- 移除Path中的冗余项

**二 拥抱Files类：**

- Files.exists() 检测文件路径是否存在
- Files.createFile() 创建文件
- Files.createDirectories()和Files.createDirectory()创建文件夹
- Files.delete()方法 可以删除一个文件或目录
- Files.copy()方法可以吧一个文件从一个地址复制到另一个位置
- 获取文件属性
- 遍历一个文件夹
- Files.walkFileTree()遍历整个目录

### [六 NIO学习总结以及NIO新特性介绍](https://blog.csdn.net/a953713428/article/details/64907250)

- **内存映射：**

这个功能主要是为了提高大文件的读写速度而设计的。内存映射文件(memory-mappedfile)能让你创建和修改那些大到无法读入内存的文件。有了内存映射文件，你就可以认为文件已经全部读进了内存，然后把它当成一个非常大的数组来访问了。将文件的一段区域映射到内存中，比传统的文件处理速度要快很多。内存映射文件它虽然最终也是要从磁盘读取数据，但是它并不需要将数据读取到OS内核缓冲区，而是直接将进程的用户私有地址空间中的一部分区域与文件对象建立起映射关系，就好像直接从内存中读、写文件一样，速度当然快了。

### [七 Java NIO AsynchronousFileChannel异步文件通](http://wiki.jikexueyuan.com/project/java-nio-zh/java-nio-asynchronousfilechannel.html)

Java7中新增了AsynchronousFileChannel作为nio的一部分。AsynchronousFileChannel使得数据可以进行异步读写。

### [八 高并发Java（8）：NIO和AIO](http://www.importnew.com/21341.html)



## 推荐阅读

### [在 Java 7 中体会 NIO.2 异步执行的快乐](https://www.ibm.com/developerworks/cn/java/j-lo-nio2/index.html)

### [Java AIO总结与示例](https://blog.csdn.net/x_i_y_u_e/article/details/52223406)

