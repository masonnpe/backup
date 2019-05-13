---
title: BlockingQueue阻塞队列
categories: 并发编程
tags: [并发容器]
---

## 基本操作

BlockingQueue继承Queue接口

| 方法                                        | 解释                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| **add()**                                   | 增加一个元素，如果队列已满，则抛出一个IIIegaISlabEepeplian异常 |
| **offer()**                                 | 添加一个元素并返回true，如果队列已满，则返回false            |
| **put()**                                   | 添加一个元素，如果队列满，则阻塞                             |
| **offer(E e, long timeout, TimeUnit unit)** | 往队列插入元素，队列满时，插入数据的线程会阻塞，直到队列有空余位置，超过给定的时间，插入的线程会退出 |
| **poll()**                                  | 移除并返问队列头部的元素，队列为空返回null                   |
| **remove()**                                | 从队列中删除数据，成功返回true，失败返回false                |
| **element()**                               | 返回队列头部的元素，如果队列为空，则抛出一个NoSuchElementException异常 |
| **peek()**                                  | 返回队列头部的元素，队列为空返回null                         |
| **take()**                                  | 移除并返回队列头部的元素，如果队列为空，则阻塞               |
| **poll(long timeout, TimeUnit unit)**       | 取出队头元素，队列为空时，线程会阻塞，超过给定的时间，线程会退出 |

<!--more-->

## 常见的阻塞队列

### ArrayBlockingQueue

**数组**实现的有界阻塞队列，一旦创建，容量不能改变。当队列容量满时，插入元素会阻塞，队列为空时，获得元素也会阻塞。默认情况下非公平，不能保证线程访问队列的公平性，访问ArrayBlockingQueue的顺序不是遵守严格的时间顺序，有可能存在，一旦ArrayBlockingQueue可以被访问时，长时间阻塞的线程依然无法访问到队列

**如果保证公平性，通常会降低吞吐量**，初始化时除了设置队列的大小还要设为true

#### 主要变量

```java
/** The queued items */
    final Object[] items;

    /** items index for next take, poll, peek or remove */
    int takeIndex;

    /** items index for next put, offer, or add */
    int putIndex;

    /** Number of elements in the queue */
    int count;

    /*
     * Concurrency control uses the classic two-condition algorithm
     * found in any textbook.
     */

    /** Main lock guarding all access */
    final ReentrantLock lock;

    /** Condition for waiting takes */
    private final Condition notEmpty;

    /** Condition for waiting puts */
    private final Condition notFull;
```

为了保证线程安全，采用的是`ReentrantLock lock`，为了保证可阻塞式的插入删除数据利用的是`Condition`，当获取数据的消费者线程被阻塞时会将该线程放置到`notEmpty`等待队列中，当插入数据的生产者线程被阻塞时，会将该线程放置到`notFull`等待队列中

#### 主要方法

**put()**

```
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
		//如果当前队列已满，将线程移入到notFull等待队列中
        while (count == items.length)
            notFull.await();
		//满足插入数据的要求，直接进行入队操作
        enqueue(e);
    } finally {
        lock.unlock();
    }
}
```

**enqueue()**

```
private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
	//插入数据,即往数组中添加数据
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;
	//通知消费者线程，当前队列中有数据可供消费
    notEmpty.signal();
}
```

**take()**

```
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
		//如果队列为空，没有数据，将消费者线程移入等待队列中
        while (count == 0)
            notEmpty.await();
		//获取数据
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

**dequeue()**

```
private E dequeue() {
    // assert lock.getHoldCount() == 1;
    // assert items[takeIndex] != null;
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
	//获取队列中的数据，即获取数组中的数据元素
    E x = (E) items[takeIndex];
    items[takeIndex] = null;
    if (++takeIndex == items.length)
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();
    //通知notFull等待队列中的线程，使其由等待队列移入到同步队列中，使其能够有机会获得lock
	notFull.signal();
    return x;
}
```

可以看出`put()`和`take()`主要是通过`condition`的通知机制来完成可阻塞式的插入数据和获取数据

### LinkedBlockingQueue

**链表**实现的有界阻塞队列，与ArrayBlockingQueue相比起来具有更高的吞吐量，初始化时需要指定大小，如果未指定，容量等于Integer.MAX_VALUE

#### 主要变量

```java
/** The capacity bound, or Integer.MAX_VALUE if none */
    private final int capacity;

    /** Current number of elements */
    private final AtomicInteger count = new AtomicInteger();

    /**
     * Head of linked list.
     * Invariant: head.item == null
     */
    transient Node<E> head;

    /**
     * Tail of linked list.
     * Invariant: last.next == null
     */
    private transient Node<E> last;

    /** Lock held by take, poll, etc */
    private final ReentrantLock takeLock = new ReentrantLock();

    /** Wait queue for waiting takes */
    private final Condition notEmpty = takeLock.newCondition();

    /** Lock held by put, offer, etc */
    private final ReentrantLock putLock = new ReentrantLock();

    /** Wait queue for waiting puts */
    private final Condition notFull = putLock.newCondition();
```

插入数据和删除数据时分别是由两个不同的lock（`takeLock`和`putLock`）来控制线程安全的，**两个对应的condition**（`notEmpty`和`notFull`）来实现可阻塞的插入和删除数据，可以降低线程由于线程无法获取到lock而进入WAITING状态的可能性，从而提高了线程并发执行的效率

#### 主要方法

**put()**

```
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    // Note: convention in all put/take/etc is to preset local var
    // holding count negative to indicate failure unless set.
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
        /*
         * Note that count is used in wait guard even though it is
         * not protected by lock. This works because count can
         * only decrease at this point (all other puts are shut
         * out by lock), and we (or some other waiting put) are
         * signalled if it ever changes from capacity. Similarly
         * for all other uses of count in other wait guards.
         */
		//如果队列已满，则阻塞当前线程，将其移入等待队列
        while (count.get() == capacity) {
            notFull.await();
        }
		//入队操作，插入数据
        enqueue(node);
        c = count.getAndIncrement();
		//若队列满足插入数据的条件，则通知被阻塞的生产者线程
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
}
```

**take()**

```
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
		//当前队列为空，则阻塞当前线程，将其移入到等待队列中，直至满足条件
        while (count.get() == 0) {
            notEmpty.await();
        }
		//移除队头元素，获取数据
        x = dequeue();
        c = count.getAndDecrement();
        //如果当前满足移除元素的条件，则通知被阻塞的消费者线程
		if (c > 1)
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();
    return x;
}
```

### LinkedBlockingDeque

链表实现的的有界阻塞双端队列

基本操作可以分为四种类型

- 特殊情况，抛出异常
- 特殊情况，返回特殊值如null或者false
- 当线程不满足操作条件时，线程会被阻塞直至条件满足
- 操作具有超时特性

### PriorityBlockingQueue

支持优先级的无界阻塞队列，默认情况下元素采用自然顺序进行排序，也可以通过实现Comparable接口的compareTo()方法来指定元素排序规则，或者初始化时通过构造器参数Comparator来指定排序规则

### SynchronousQueue

因为只有线程在删除数据时，其他线程才能插入数据。如果当前有线程在插入数据时，线程才能删除数据

### LinkedTransferQueue

链表实现的无界阻塞队列，实现了TransferQueue接口

**transfer(E e)**
如果当前有消费线程想要获得队头元素而阻塞时，生产线程可以调用transfer方法将数据传递给消费线程。如果当前没有消费线程的话，生产线程就会将数据插入到队尾，**直到有消费线程进行消费才退出**

**tryTransfer(E e)**
如果当前有消费线程正在消费数据，该方法可以将数据立即传送给消费线程，如果当前没有消费者线程消费数据的话，就**立即返回**false

**tryTransfer(E e,long timeout,imeUnit unit)**
如果在规定的时间内，数据没有被消费线程进行消费的话，就返回false

### DelayQueue

实现Delayed接口的无界阻塞队列，只有当数据对象的延时时间达到时才能插入到队列进行存储。通过Delayed接口的`getDelay(TimeUnit.NANOSECONDS)`来进行判定，如果该方法返回的值小于等于0则说明该数据元素的延时期已满

## 实现生产者消费者

生产者线程生产数据，消费者线程消费数据，为了解耦生产者和消费者的关系，通常会采用共享的数据区域，生产者生产数据之后直接放置在共享数据区中，这个共享数据区域中应该具备这样的线程间并发协作的功能

- 如果共享数据区已满的话，阻塞生产者继续生产数据放置入内
- 如果共享数据区为空的话，阻塞消费者继续消费数据



##### 前面介绍了各种队列实现，在日常的应用开发中，如何进行选择呢？

以LinkedBlockingQueue、ArrayBlockingQueue和SynchronousQueue为例，我们一起来分析一下，根据需求可以从很多方面考量：
考虑应用场景中对队列边界的要求。ArrayBlockingQueue是有明确的容量限制的，而LinkedBlockingQueue则取决于我们是否在创建时指定，SynchronousQueue则干脆不能缓存任何元素。
从空间利用角度，数组结构的ArrayBlockingQueue要比LinkedBlockingQueue紧凑，因为其不需要创建所谓节点，但是其初始分配阶段就需要一段连续的空间，所以初始内存需求更大。
通用场景中，LinkedBlockingQueue的吞吐量一般优于ArrayBlockingQueue，因为它实现了更加细粒度的锁操作。
ArrayBlockingQueue实现比较简单，性能更好预测，属于表现稳定的“选手”。
如果我们需要实现的是两个线程之间接力性（handof）的场景，按照专栏上一讲的例子，你可能会选择CountDownLatch，但是SynchronousQueue也是完美符合这种场景的，而且线程间协调和数据传输统一起来，代码更加规范。
可能令人意外的是，很多时候SynchronousQueue的性能表现，往往大大超过其他实现，尤其是在队列元素较小的场景。


