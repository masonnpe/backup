---
title: ThreadPoolExecutor线程池
categories: 并发编程
tags: [线程池]
---

在《阿里巴巴Java开发手册》“并发处理”这一章节，明确指出线程资源必须通过线程池提供，不允许在应用中自行显示创建线程。

**为什么要使用线程池？** 

- 线程是稀缺资源，创建开销大，不能频繁的创建，而线程池可以重复利用线程减少开销，提升系统响应速度，减去创建线程的时间
- 解耦，线程的创建、执行完全分开，便于维护

**线程池定义的状态**

```java
// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;// 运行状态 可以接受队列里的任务
private static final int SHUTDOWN   =  0 << COUNT_BITS;// 不在接受新的任务，且队列里的任务要执行完
private static final int STOP       =  1 << COUNT_BITS;// 不再接受新任务，且队列里的任务和正在执行的任务都中断
private static final int TIDYING    =  2 << COUNT_BITS;// 所有任务都执行完毕
private static final int TERMINATED =  3 << COUNT_BITS;// 终止状态
```

<!--more-->

**工作队列**

- `PriorityBlockingQueue `优先队列，优先级低的可能永远不能被执行
- `ArrayBlockingQueue`基于数组的先进先出队列，此队列创建时必须指定大小； 
- `LinkedBlockingQueue`基于链表的先进先出队列，如果创建时没有指定，则默认为Integer.MAX_VALUE； 
- `SynchronousQueue`这个队列不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。 

一般如果线程池任务队列采用LinkedBlockingQueue队列的话，那么不会拒绝任何任务（因为队列大小没有限制），这种情况下，ThreadPoolExecutor最多仅会按照最小线程数来创建线程，也就是说线程池大小被忽略了。

如果线程池任务队列采用ArrayBlockingQueue队列的话，那么ThreadPoolExecutor将会采取一个非常负责的算法，比如假定线程池的最小线程数为4，最大为8所用的ArrayBlockingQueue最大为10。随着任务到达并被放到队列中，线程池中最多运行4个线程（即最小线程数）。即使队列完全填满，也就是说有10个处于等待状态的任务，ThreadPoolExecutor也只会利用4个线程。如果队列已满，而又有新任务进来，此时才会启动一个新线程，这里不会因为队列已满而拒接该任务，相反会启动一个新线程。新线程会运行队列中的第一个任务，为新来的任务腾出空间。

**拒绝处理任务时的策略**

- `ThreadPoolExecutor.AbortPolicy`:直接拒绝所提交的任务并抛出`RejectedExecutionException`异常
- `ThreadPoolExecutor.DiscardPolicy`：不处理直接丢弃掉任务，不抛出异常。 
- `ThreadPoolExecutor.DiscardOldestPolicy`：丢弃掉阻塞队列中存放时间最久的任务，执行当前任务
- `ThreadPoolExecutor.CallerRunsPolicy`：只用调用者所在的线程来执行任务

## 主要方法

**execute()**

```java
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
    	int c = ctl.get(); // 获得线程池状态
        if (workerCountOf(c) < corePoolSize) {// 小于线程池大小创建一个新的线程运行
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {// 线程在运行，且成功进入阻塞队列
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))// 双重校验，线程状态，如果线程状态变了，就
                reject(command);					  // 从队列移除任务
            else if (workerCountOf(recheck) == 0)// 线程池为空，创建一个新的线程执行
                addWorker(null, false);
        }
        else if (!addWorker(command, false)) // 尝试新建线程，失败就执行拒绝策略
            reject(command);
    }
```

**submit() **

也是用来向线程池提交任务的，但是它和execute()方法不同，它能够返回任务执行的结果 ，利用了Future来获取任务执行结果 

**shutdown()**

用来关闭线程池的 如果调用了shutdown()方法，则线程池处于SHUTDOWN状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕； 

**shutdownNow()**

如果调用了shutdownNow()方法，则线程池处于STOP状态，此时线程池不能接受新的任务，并且会去尝试停止所有的正在执行和未执行任务的线程，并返回等待执行任务的列表

## 线程池任务执行流程

![image](https://wx2.sinaimg.cn/large/007iUdjSly1g0l1ka7jfij30sg0jq76i.jpg)

## Executors 线程池种类

**newCachedThreadPool**

- 底层：返回ThreadPoolExecutor实例，corePoolSize为0；maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为60L；unit为TimeUnit.SECONDS；workQueue为SynchronousQueue(同步队列)
- 通俗：当有新任务到来，则插入到SynchronousQueue中，由于SynchronousQueue是同步队列，因此会在池中寻找可用线程来执行，若有可以线程则执行，若没有可用线程则创建一个线程来执行该任务；若池中线程空闲时间超过指定大小，则该线程会被销毁。
- 适用：执行很多短期异步的小程序

**newFixedThreadPool**

- 底层：返回ThreadPoolExecutor实例，接收参数为所设定线程数量nThread，corePoolSize为nThread，maximumPoolSize为nThread；keepAliveTime为0L(不限时)；unit为：TimeUnit.MILLISECONDS；WorkQueue为：new LinkedBlockingQueue<Runnable>() 无界阻塞队列
- 通俗：创建可容纳固定数量线程的池子，每隔线程的存活时间是无限的，当池子满了就不在添加线程了；如果池中的所有线程均在繁忙状态，对于新任务会进入阻塞队列中(无界的阻塞队列)
- 适用：执行长期的任务，性能好很多，适用于可以预测线程数量的业务中，或者服务器负载较重，对当前线程数量进行限制。

**newSingleThreadExecutor**

- 底层：FinalizableDelegatedExecutorService包装的ThreadPoolExecutor实例，corePoolSize为1；maximumPoolSize为1；keepAliveTime为0L；unit为：TimeUnit.MILLISECONDS；workQueue为：new LinkedBlockingQueue<Runnable>() 无解阻塞队列
- 通俗：创建只有一个线程的线程池，且线程的存活时间是无限的；当该线程正繁忙时，对于新任务会进入阻塞队列中(无界的阻塞队列)
- 适用：一个任务一个任务执行的场景

**newScheduledThreadPool**

- 底层：创建ScheduledThreadPoolExecutor实例，corePoolSize为传递来的参数，maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为0；unit为：TimeUnit.NANOSECONDS；workQueue为：new DelayedWorkQueue() 一个按超时时间升序排序的队列
- 通俗：创建一个固定大小的线程池，线程池内线程存活时间无限制，线程池可以支持定时及周期性任务执行，如果所有线程均处于繁忙状态，对于新任务会进入DelayedWorkQueue队列中，这是一种按照超时时间排序的队列结构
- 适用：周期性执行任务的场景

**newWorkStealingPool**

- 获取当前可用的线程数量进行创建作为并行级别
- 使用ForkJoinPool
- 使用一个无限队列来保存需要执行的任务，可以传入线程的数量，不传入，则默认使用当前计算机中可用的cpu数量，使用分治法来解决问题，使用fork()和join()来进行调用
- 适用：适用于大耗时的操作，可以并行来执行

## 合理配置线程池参数

《阿里巴巴Java开发手册》中强制线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，明确线程池的运行规则，规避资源耗尽的风险

Executors 返回线程池对象的弊端如下：

- **FixedThreadPool 和 SingleThreadExecutor** ： 允许请求的队列长度为 Integer.MAX_VALUE,如果工作线程数目太少，导致处理跟不上入队的速度，这就很有可能占用大量系统内存，可能堆积大量的请求，从而导致OOM。诊断时，可以使用jmap之类的工具，查看是否有大量的任务对象入队。
- **CachedThreadPool 和 ScheduledThreadPool** ： 允许创建的线程数量为 Integer.MAX_VALUE ，通常在处理大量短时任务时，使用缓存的线程池可能会创建大量线程，如果线程数目不断增长（可以使用jstack等工具检查）因为任务逻辑有问题，导致工作线程迟迟不能被释放（线程泄漏），从而导致OOM。

要想合理的配置线程池，就必须首先分析任务特性，可以从以下几个角度来进行分析

**任务性质不同**：**CPU密集型**任务配置尽可能少的线程数量，如配置**cpu+1**个线程的线程池，减少上下文切换。**IO密集型任务**则由于需要等待IO操作，线程并不是一直在执行任务，则配置尽可能多的线程，如**cpu x 2**个，让CPU处理更多的业务。**混合型的任务**，如果可以拆分，则将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐率要高于串行执行的吞吐率，如果这两个任务执行时间相差太大，则没必要进行分解。可以通过`Runtime.getRuntime().availableProcessors()`方法获得当前设备的CPU个数

**优先级不同的任务**：可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先得到执行，需要注意的是如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行

**任务的执行时间**：执行时间不同的任务可以交给不同规模的线程池来处理，或者也可以使用优先级队列，让执行时间短的任务先执行

**任务的依赖性**：任务依赖其他系统资源，如数据库连接，因为线程提交SQL后需要等待数据库返回结果，如果等待的时间越长CPU空闲时间就越长，那么线程数应该设置越大，这样才能更好的利用CPU

**如果提交任务时，线程池队列已满**

- 如果使用的是无界队列LinkedBlockingQueue，也就是无界队列的话，没关系，继续添加任务到阻塞队列中等待执行，因为LinkedBlockingQueue可以近乎认为是一个无穷大的队列，可以无限存放任务
- 如果使用的是有界队列比如ArrayBlockingQueue，任务首先会被添加到ArrayBlockingQueue中，ArrayBlockingQueue满了，会根据maximumPoolSize的值增加线程数量，如果增加了线程数量还是处理不过来，ArrayBlockingQueue继续满，那么则会使用拒绝策略RejectedExecutionHandler处理满了的任务，默认是AbortPolicy

注意避免死锁等同步问题，对于死锁的场景和排查
尽量避免在使用线程池时操作ThreadLocal

## 推荐阅读

[线程池详解](http://www.ideabuffer.cn/2017/04/04/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E7%BA%BF%E7%A8%8B%E6%B1%A0%EF%BC%9AThreadPoolExecutor/)

http://www.cnblogs.com/dolphin0520/p/3932921.html

对于线程池感兴趣的可以查看我的这篇文章：[《Java多线程学习（八）线程池与Executor 框架》](http://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247484042&idx=1&sn=541dbf2cb969a151d79f4a4f837ee1bd&chksm=fd9854ebcaefddfd1876bb96ab218be3ae7b12546695a403075d4ed22e5e17ff30ebdabc8bbf#rd) 点击阅读原文即可查看到该文章的最新版。、[深入理解Java线程池](http://www.ideabuffer.cn/2017/04/04/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E7%BA%BF%E7%A8%8B%E6%B1%A0%EF%BC%9AThreadPoolExecutor/)