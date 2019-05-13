---
title: ConcurrentHashMap的工作原理及代码实现
categories: 并发编程
tags: [ConcurrentHashMap]
---

1.8以后的锁的颗粒度，是加在链表头上的，这个是个思路上的突破。



## 为什么需要ConcurrentHashMap

Hashtable本身比较低效，因为它的实现基本就是将put、get、size等各种方法加上synchronized。这就导致了所有并发操作都要竞争同一把锁，大大降低了并发操作的效率。
HashMap不是线程安全的，并发情况会导致类似CPU占用100%等一些问题。Collections提供的同步包装器只是利用输入Map构造了另一个同步版本，所有操作虽然不再声明成为synchronized方法，但是还是利用了“this”作为互斥的mutex，没有真正意义上的改进！
所以Hashtable或者同步包装版本，都只是适合在非高度并发的场景下。

<!--more-->

## ConcurrentHashMap分析

早期ConcurrentHashMap，其实现是基于分段锁，也就是将内部进行分段（Segment），里面则是HashEntry的数组，和HashMap类似，哈希相同的条目也是以链表形式存放。HashEntry内部使用volatile的value字段来保证可见性，也利用了不可变对象的机制以改进利用Unsafe提供的底层能力，比如volatile access，去直接完成部分操作，以最优化性能，毕竟Unsafe中的很多操作都是JVM intrinsic优化过的。核心是利用分段设计，在进行并发操作的时候，只需要锁定相应段，这样就有效避免了类似Hashtable整体同步的问题，大大提高了性能。
对于put操作，首先是通过二次哈希避免哈希冲突，然后以Unsafe调用方式，直接获取相应的Segment，然后进行线程安全的put操作：
所以，从上面的源码清晰的看出，在进行并发写操作时：
ConcurrentHashMap会获取冲入锁，以保证数据一致性，Segment本身就是基于ReentrantLock的扩展实现，所以，在并发修改期间，相应Segment是被锁定的。
如果不进行同步，简单的计算所有Segment的总值，可能会因为并发put，导致结果不准确，但是直接锁定所有Segment进行计算，就会变得非常昂贵。其实，分离锁也限制了Map的初始化等操作。
所以，ConcurrentHashMap的实现是通过重试机制（RETRIES_BEFORE_LOCK，指定重试次数2），来试图获得可靠值。如果没有监控到发生变化（通过对比Segment.modCount），就直接返回，否则获取锁进行操作。
下面我来对比一下，在Java 8中ConcurrentHashMap
数据存储利用volatile来保证可见性。
使用CAS等操作，在特定场景进行无锁并发操作。
使用Unsafe、LongAdder之类底层手段，进行极端情况的优化。
先看看现在的数据存储内部实现，我们可以发现Key是fnal的，因为在生命周期中，一个条目的Key发生变化是不可能的；与此同时val，则声明为volatile，以保证可见性。
直接看并发的put是如何实现的。
fnal V putVal(K key, V value, boolean onlyIfAbsent) { if (key == null || value == null) throw new NullPointerException();
int hash = spread(key.hashCode());
int binCount = 0;
for (Node[] tab = table;;) {
Node f; int n, i, fh; K fk; V fv;
if (tab == null || (n = tab.length) == 0)
tab = initTable();
else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
// 利用CAS去进行无锁线程安全操作，如果bin是空的
if (casTabAt(tab, i, null, new Node(hash, key, value)))
break;
}
else if ((fh = f.hash) == MOVED)
tab = helpTransfer(tab, f);
else if (onlyIfAbsent // 不加锁，进行检查
&& fh == hash
&& ((fk = f.key) == key || (fk != null && key.equals(fk)))
&& (fv = f.val) != null)
return fv;
else {
V oldVal = null;
synchronized (f) {
// 细粒度的同步修改操作...
}
}
// Bin超过阈值，进行树化
if (binCount != 0) {
if (binCount >= TREEIFY_THRESHOLD)
treeifyBin(tab, i);
if (oldVal != null)
return oldVal;
break;
}
}
}
addCount(1L, binCount);
return null;
}
初始化操作实现在initTable里面，这是一个典型的CAS使用场景，利用volatile的sizeCtl作为互斥手段：如果发现竞争性的初始化，就spin在那里，等待条件恢复；否则利用CAS设
置排他标志。如果成功则进行初始化；否则重试。
请参考下面代码：
private fnal Node[] initTable() {
Node[] tab; int sc;
while ((tab = table) == null || tab.length == 0) {
// 如果发现冲突，进行spin等待
if ((sc = sizeCtl) < 0)
Thread.yield();
// CAS成功返回true，则进入真正的初始化逻辑
else if (U.compareAndSetInt(this, SIZECTL, sc, -1)) {
try {
if ((tab = table) == null || tab.length == 0) {
int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
@SuppressWarnings("unchecked")
Node[] nt = (Node[])new Node[n];
table = tab = nt;
sc = n - (n >>> 2);
}
} fnally {
sizeCtl = sc;
}
break;
}
}
return tab;
}
当bin为空时，同样是没有必要锁定，也是以CAS操作去放置。
你有没有注意到，在同步逻辑上，它使用的是synchronized，而不是通常建议的ReentrantLock之类，这是为什么呢？现代JDK中，synchronized已经被不断优化，可以不再过分
极客时间
担心性能差异，另外，相比于ReentrantLock，它可以减少内存消耗，这是个非常大的优势。
与此同时，更多细节实现通过使用Unsafe进行了优化，例如tabAt就是直接利用getObjectAcquire，避免间接调用的开销。
satic fnal  Node tabAt(Node[] tab, int i) {
return (Node)U.getObjectAcquire(tab, ((long)i << ASHIFT) + ABASE);
}
再看看，现在是如何实现size操作的。阅读代码你会发现，真正的逻辑是在sumCount方法中， 那么sumCount做了什么呢？
fnal long sumCount() {
CounterCell[] as = counterCells; CounterCell a;
long sum = baseCount;
if (as != null) {
for (int i = 0; i < as.length; ++i) {
if ((a = as[i]) != null)
sum += a.value;
}
}
return sum;
}
我们发现，虽然思路仍然和以前类似，都是分而治之的进行计数，然后求和处理，但实现却基于一个奇怪的CounterCell。 难道它的数值，就更加准确吗？数据一致性是怎么保证
的？
satic fnal class CounterCell {
volatile long value;
CounterCell(long x) { value = x; }
}
其实，对于CounterCell的操作，是基于java.util.concurrent.atomic.LongAdder进行的，是一种JVM利用空间换取更高效率的方法，利用了Striped64内部的复杂逻辑。这个东
西非常小众，大多数情况下，建议还是使用AtomicLong，足以满足绝大部分应用的性能需求。
今天我从线程安全问题开始，概念性的总结了基本容器工具，分析了早期同步容器的问题，进而分析了Java 7和Java 8中ConcurrentHashMap是如何设计实现的，希
望ConcurrentHashMap的并发技巧对你在日常开发可以有所帮助。