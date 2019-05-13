---
title: CopyOnWrite
categories: 并发编程
tags: [并发容器]
---

可以对CopyOnWrite容器进行并发的读，而不需要加锁。CopyOnWrite并发容器用于读多写少的并发场景。比如白名单，黑名单，商品类目的访问和更新场景，数据一致性要求不高的场景。

## CopyOnWriteArrayList

如果想list的读效率更高的话，保证读线程无论什么时候都不被阻塞，使用**CopyOnWriteArrayList**容器，写时复制的思想来通过延时更新的策略，放弃数据实时性实现数据的最终一致性，并且能够保证读线程间不阻塞，往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，往新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。对CopyOnWrite容器进行并发的读的时候，不需要加锁，因为当前容器不会添加任何元素

<!--more-->

## CopyOnWriteArrayList

### 适用场景

CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。

但是 CopyOnWriteArrayList 有其缺陷：

- 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；可能会造成频繁的minor GC和major GC
- 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。

所以 CopyOnWriteArrayList 不适合内存敏感以及对实时性要求很高的场景。

## CopyOnWriteArrayList实现原理

**add()**

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
	//1. 使用Lock,保证写线程在同一时刻只有一个，防止多次复制
    lock.lock();
    try {
		//2. 获取旧数组引用
        Object[] elements = getArray();
        int len = elements.length;
		//3. 创建新的数组，并将旧数组的数据复制到新数组中
        Object[] newElements = Arrays.copyOf(elements, len + 1);
		//4. 往新数组中添加新的数据	        
		newElements[len] = e;
		//5. 将旧数组引用指向新的数组
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

Arrays.copyof()用于复制指定的数组内容以达到扩容的目的，该方法对不同的基本数据类型都有对应的重载方法 

## CopyOnWrite与ReadWriteLock对比

`CopyOnWrite`和`ReadWriteLock`都是通过读写分离的思想实现，读线程之间互不阻塞

**ReadWriteLock**：对读线程而言，为了实现数据实时性，在写锁被获取后，读线程会等待或者当读锁被获取后，写线程会等待。

**CopyOnWrite**：牺牲数据实时性而保证数据最终一致性，即读线程对数据的更新是延时感知的，读线程不会存在等待的情况