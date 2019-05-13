---
title: GC日志
categories: JVM
tags: [JVM,GC]
---

垃圾收集器长时间停顿，表现在 Web 页面上可能是页面响应码 500 之类的服务器错误问题，如果是个支付过程可能会导致支付失败，将造成公司的直接经济损失。

### MetaSpace内存溢出

`JDK8` 使用 `MetaSpace` 来保存类加载之后的类信息，字符串常量池也被移动到 Java 堆

JDK 8 中将类信息移到到了本地堆内存(Native Heap)中，将原有的永久代移动到了本地堆中成为 `MetaSpace` ,如果不指定该区域的大小，JVM 将会动态的调整。

可以使用 `-XX:MaxMetaspaceSize=10M` 来限制最大元数据。这样当不停的创建类时将会占满该区域并出现 `OOM`

动态代理对象太多也会 oom动态代理生成的对象在Jvm中指向的不是同一个地址，它只是与源对象有相同的hashcode值而已

<!--more-->

## CMS (concurrent mode failure)

- 老年代碎片化严重，无法容纳新生代提升上来的大对象
- 新生代来不及回收，老年代被用完

发送这种情况，应用线程将会全部停止（相当于网站这段时间无法响应用户请求），进行压缩式垃圾收集（回退到 Serial Old 算法）

解决办法：

- 新生代提升过快问题：（1）如果频率太快的话，说明空间不足，首先可以尝试调大新生代空间和晋升阈值。（2）如果内存有限，可以设置 CMS 垃圾收集在老年代占比达到多少时启动来减少问题发生频率（越早启动问题发生频率越低，但是会降低吞吐量，具体得多调整几次找到平衡点），参数如下：如果没有第二个参数，会随着 JVM 动态调节 CMS 启动时间

-XX:CMSInitiatingOccupancyFraction=68 （默认是 68）

-XX:+UseCMSInitiatingOccupancyOnly

- 老年代碎片严重问题：（1）如果频率太快或者 Full GC 后空间释放不多的话，说明空间不足，首先可以尝试调大老年代空间（2）如果内存不足，可以设置进行 n 次 CMS 后进行一次压缩式 Full GC，参数如下：

-XX:+UseCMSCompactAtFullCollection：允许在 Full GC 时，启用压缩式 GC

-XX:CMSFullGCBeforeCompaction=n     在进行 n 次，CMS 后，进行一次压缩的 Full GC，用以减少 CMS 产生的碎片

## CMS (promotion failed)

在 Minor GC 过程中，Survivor Unused 可能不足以容纳 Eden 和另一个 Survivor 中的存活对象， 那么多余的将被移到老年代， 称为过早提升（Premature Promotion）。 这会导致老年代中短期存活对象的增长， 可能会引发严重的性能问题。  再进一步， 如果老年代满了， Minor GC 后会进行 Full GC， 这将导致遍历整个堆， 称为提升失败（Promotion Failure）。

提升失败日志：  

提升失败原因：Minor GC 时发现 Survivor 空间放不下，而老年代的空闲也不够

- 新生代提升太快
- 老年代碎片太多，放不下大对象提升（表现为老年代还有很多空间但是，出现了 promotion failed）

解决方法：是调整年轻代和年老代的比例，还有CMSGC的时机

​       两条和上面 concurrent mode failure 一样

​       另一条，是因为 Survivor Unused 不足，那么可以尝试调大 Survivor 来尝试下 

三. 在 GC 的时候其他系统活动影响

有些时候系统活动诸如内存换入换出（vmstat）、网络活动（netstat）、I/O （iostat）在 GC 过程中发生会使 GC 时间变长。

前提是你的服务器上是有 SWAP 区域（用 top、 vmstat 等命令可以看出）用于内存的换入换出，那么操作系统可能会将 JVM 中不活跃的内存页换到 SWAP 区域用以释放内存给线程使用（这也透露出内存开始不够用了）。内存换入换出是一个开销巨大的磁盘操作，比内存访问慢好几个数量级。

看一段 GC 日志：耗时 29.47 秒 

再看看此时的 vmstat 命令中 si、so 列的数值，如果数值大说明换入换出严重，这是内存不足的表现。

解决方法：减少线程，这样可以降低内存换入换出；增加内存；如果是 JVM 内存设置过大导致线程所用内存不足，则适当调低 -Xmx 和 -Xms。

五. 总结

​       长时间停顿问题的排查及解决首先需要一定的信息和方法论：

- 详细的 GC 日志
- 借助 Linux 平台下的 iostat、vmstat、netstat、mpstat 等命令监控系统情况
- 查看 GC 日志中是否出现了上述的典型内存异常问题（promotion failed, concurrent mode failure），整体来说把上述两个典型内存异常情况控制在可接受的发生频率即可，对 CMS 碎片问题来说杜绝以上问题似乎不太可能，只能靠 G1 来解决了
- 是不是 JVM 本身的 bug 导致的
- 如果程序没问题，参数调了几次还是不能解决，可能说明流量太大，需要加机器把压力分散到更多 JVM 上

## gc常见错误

**java.lang.OutOfMemoryError: Java heap space**

原因：Heap内存溢出，意味着Young和Old generation的内存不够。

解决：调整java启动参数-Xms -Xmx 来增加Heap内存。

**java.lang.OutOfMemoryError: unable to create new native thread**

原因：Stack空间不足以创建额外的线程，要么是创建的线程过多，要么是Stack空间确实小了。

解决：由于JVM没有提供参数设置总的stack空间大小，但可以设置单个线程栈的大小；而系统的用户空间一共是3G，除了Text/Data/BSS /MemoryMapping几个段之外，Heap和Stack空间的总量有限，是此消彼长的。因此遇到这个错误，可以通过两个途径解决：1.通过 -Xss启动参数减少单个线程栈大小，这样便能开更多线程（当然不能太小，太小会出现StackOverflowError）；2.通过-Xms -Xmx 两参数减少Heap大小，将内存让给Stack（前提是保证Heap空间够用）。

**java.lang.OutOfMemoryError: Requested array size exceeds VM limit**

原因：这个错误比较少见（试着new一个长度1亿的数组看看），同样是由于Heap空间不足。如果需要new一个如此之大的数组，程序逻辑多半是不合理的。

解决：修改程序逻辑吧。或者也可以通过-Xmx来增大堆内存。

在GC花费了大量时间，却仅回收了少量内存时，也会报出OutOfMemoryError ，我只遇到过一两次。当使用-XX:+UseParallelGC或-XX:+UseConcMarkSweepGC收集器时，在上述情况下会报错，在 HotSpot GC Turning文档 上有说明：

The parallel(concurrent) collector will throw an OutOfMemoryError if too much time is being spent in garbage collection: if more than 98% of the total time is spent in garbage collection and less than 2% of the heap is recovered, an OutOfMemoryError will be thrown.

对这个问题，一是需要进行GC turning，二是需要优化程序逻辑。

**java.lang.StackOverflowError**

原因：这也内存溢出错误的一种，即线程栈的溢出，要么是方法调用层次过多（比如存在无限递归调用），要么是线程栈太小。

解决：优化程序设计，减少方法调用层次；调整-Xss参数增加线程栈大小。

**IOException: Too many open files**

原因： 这个是由于TCP connections 的buffer 大小不够用了。

**java.lang.OutOfMemoryError:Direct buffer memory**

解决：调整-XX:MaxDirectMemorySize=





