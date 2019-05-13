---
title: Queue
categories: Java
tags: [Queue]
---

Queue是一个FIFO的数据结构，Queue接口与List、Set一样都继承了Collection接口

Deque双向队列，队列两端的元素既能入队也能出队，LinkedList实现了Deque接口

## 非阻塞队列

- PriorityQueue实质上维护了一个有序列表。加入到 Queue 中的元素根据它们的天然排序（通过其 java.util.Comparable 实现）或者根据传递给构造函数的 java.util.Comparator 实现来定位  
- ConcurrentLinkedQueue是基于链接节点的、**线程安全**的**无界**队列。并发访问不需要同步，需要遍历队列。  因为它在队列的尾部添加元素并从头部删除它们，所以只要不需要知道队列的大小，ConcurrentLinkedQueue 对公共集合的共享访问就可以工作得很好。收集关于队列大小的信息会很慢，需要遍历队列。采用CAS机制（compareAndSwapObject原子操作）。不支持阻塞去取元素

## 阻塞队列

- LinkedBlockingQueue ：一个由链接节点支持的可选有界队列。LinkedBlockingQueue的容量是没有上限的（说的不准确，在不指定时容量为Integer.MAX_VALUE，不要然的话在put时怎么会受阻呢），但是也可以选择指定其最大容量，它是基于链表的队列，此队列按 FIFO（先进先出）排序元素。支持阻塞的take()方法 。使用 ReentrantLock 锁,添加元素为原子操作的队列

- ArrayBlockingQueue ：一个由**数组**支持的**有界**队列。ArrayBlockingQueue在构造时需要指定容量， 并可以选择是否需要公平性，如果公平参数被设置true，等待时间最长的线程会优先得到处理（其实就是通过将ReentrantLock设置为true来 达到这种公平性的：即等待时间最长的线程会先操作）。通常，公平性会使你在性能上付出代价，只有在的确非常需要的时候再使用它。它是基于数组的阻塞循环队 列，此队列按 FIFO（先进先出）原则对元素进行排序。

- PriorityBlockingQueue ：一个由**优先级堆**支持的**无界**优先级队列，而不是先进先出队列。元素按优先级顺序被移除，该队列也没有上限（看了一下源码，PriorityBlockingQueue是对 PriorityQueue的再次包装，是基于堆数据结构的，而PriorityQueue是没有容量限制的，与ArrayList一样，所以在优先阻塞 队列上put时是不会受阻的。虽然此队列逻辑上是无界的，但是由于资源被耗尽，所以试图执行添加操作可能会导致 OutOfMemoryError），但是如果队列为空，那么取元素的操作take就会阻塞，所以它的检索操作take是受阻的。另外，往入该队列中的元 素要具有比较能力。

- DelayQueue ：一个由**优先级堆**支持的、基于时间的调度队列。DelayQueue（基于PriorityQueue来实现的）是一个存放Delayed 元素的无界阻塞队列，只有在延迟期满时才能从中提取元素。该队列的头部是延迟期满后保存时间最长的 Delayed 元素。如果延迟都还没有期满，则队列没有头部，并且poll将返回null。当一个元素的 getDelay(TimeUnit.NANOSECONDS) 方法返回一个小于或等于零的值时，则出现期满，poll就以移除这个元素了。此队列不允许使用 null 元素。

- SynchronousQueue （并发同步阻塞队列）一个利用 BlockingQueue 接口的简单聚集（rendezvous）机制。

<!--more-->

| method  | description                                              |
| ------- | -------------------------------------------------------- |
| add     | 添加元素  如果队列满 抛出IIIegaISlabEepeplian异常        |
| offer   | 元素插入到队尾 如果队列满，返回false                     |
| put     | 添加元素 队列满则阻塞                                    |
| peek    | 不移除元素返回队头，队列为空返回null                     |
| element | 不移除元素返回队头，队列为空抛NoSuchElementException异常 |
| poll    | 移除元素返回队头，队列为空返回null                       |
| remove  | 移除元素返回队头，队列为空抛NoSuchElementException异常   |
| take    | 移除并返回队头，队列空则阻塞                             |

## 生产者-消费者

阻塞队列支持设计模式

```java

package com.yao;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
public class BlockingQueueTest {
 /**
 定义装苹果的篮子
  */
 public static class Basket{
  // 篮子，能够容纳3个苹果
  BlockingQueue<String> basket = new ArrayBlockingQueue<String>(3);

  // 生产苹果，放入篮子
  public void produce() throws InterruptedException{
   // put方法放入一个苹果，若basket满了，等到basket有位置
   basket.put("An apple");
  }
  // 消费苹果，从篮子中取走
  public String consume() throws InterruptedException{
   // get方法取出一个苹果，若basket为空，等到basket有苹果为止
   String apple = basket.take();
   return apple;
  }

  public int getAppleNumber(){
   return basket.size();
  }

 }
 //　测试方法
 public static void testBasket() {
  // 建立一个装苹果的篮子
  final Basket basket = new Basket();
  // 定义苹果生产者
  class Producer implements Runnable {
   public void run() {
    try {
     while (true) {
      // 生产苹果
      System.out.println("生产者准备生产苹果：" 
        + System.currentTimeMillis());
      basket.produce();
      System.out.println("生产者生产苹果完毕：" 
        + System.currentTimeMillis());
      System.out.println("生产完后有苹果："+basket.getAppleNumber()+"个");
      // 休眠300ms
      Thread.sleep(300);
     }
    } catch (InterruptedException ex) {
    }
   }
  }
  // 定义苹果消费者
  class Consumer implements Runnable {
   public void run() {
    try {
     while (true) {
      // 消费苹果
      System.out.println("消费者准备消费苹果：" 
        + System.currentTimeMillis());
      basket.consume();
      System.out.println("消费者消费苹果完毕：" 
        + System.currentTimeMillis());
      System.out.println("消费完后有苹果："+basket.getAppleNumber()+"个");
      // 休眠1000ms
      Thread.sleep(1000);
     }
    } catch (InterruptedException ex) {
    }
   }
  }

  ExecutorService service = Executors.newCachedThreadPool();
  Producer producer = new Producer();
  Consumer consumer = new Consumer();
  service.submit(producer);
  service.submit(consumer);
  // 程序运行10s后，所有任务停止
  try {
   Thread.sleep(10000);
  } catch (InterruptedException e) {
  }
  service.shutdownNow();
 }
 public static void main(String[] args) {
  BlockingQueueTest.testBasket();
 }
}
```



```
1.LinkedBlockingQueue是使用锁机制，ConcurrentLinkedQueue是使用CAS算法，虽然LinkedBlockingQueue的底层获取锁也是使用的CAS算法

2.关于取元素，ConcurrentLinkedQueue不支持阻塞去取元素，LinkedBlockingQueue支持阻塞的take()方法，如若大家需要ConcurrentLinkedQueue的消费者产生阻塞效果，需要自行实现

3.关于插入元素的性能，从字面上和代码简单的分析来看ConcurrentLinkedQueue肯定是最快的，但是这个也要看具体的测试场景，我做了两个简单的demo做测试，测试的结果如下，两个的性能差不多，但在实际的使用过程中，尤其在多cpu的服务器上，有锁和无锁的差距便体现出来了，ConcurrentLinkedQueue会比LinkedBlockingQueue快很多：
```

linkedlist线程不安全