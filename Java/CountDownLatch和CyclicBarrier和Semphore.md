---
title: CountDownLatch和CyclicBarrier
categories: 并发编程
tags: [并发工具]
---

https://www.jianshu.com/p/7c7a5df5bda6?ref=myread

## CountDownLatch

`CountDownLatch`允许一个或多个线程，等待另一组线程完成操作，再继续执行。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

**构造函数**

CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1，计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。 构造器中的计数值（count）实际上就是需要等待的线程数量。这个值只能被设置**一**次，不能重置

<!--more-->

**countDown()**

通知CountDownLatch对象，已经完成了任务。每调用一次这个方法，在构造函数中初始化的count值就减1。当n个线程都调用了这个方法使count的值等于0，主线程就能从`await()`处恢复，执行自己的任务

**await()**

主线程必须在启动其他线程后调用`await()` 这样主线程就会在这个方法上阻塞，直到其他线程完成各自的任务

**await(long timeout, TimeUnit unit)**

与上面的await方法功能一致，只不过这里有了时间限制，调用该方法的线程等到指定的timeout时间后，不管count是否减至为0，都会继续往下执行

## CyclicBarrier

`CyclicBarrier`允许一组线程**相互之间等待**，直到所有线程都达到一个集合点后再继续执行

**构造函数**

CyclicBarrier(int parties)：声明需要拦截的线程数

CyclicBarrier(int parties, Runnable barrierAction)：声明需要拦截的线程数并定义一个Runnable对象，在所有线程到达集合点后，执行Runnable任务

**await()**

等待直到所有的线程都到达指定的临界点

```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
// 在CyclicBarrier中，同一批线程属于同一代。当有parties个线程到达barrier，generation就会被更新换代。其中broken标识该当前CyclicBarrier是否已经处于中断状态
private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            final Generation g = generation;

            if (g.broken)
                throw new BrokenBarrierException();
			// 如果当前线程被中断，则通过breakBarrier()终止CyclicBarrier，唤醒CyclicBarrier中所有等待线程。
            if (Thread.interrupted()) {
                breakBarrier();
                throw new InterruptedException();
            }
			// 计数器-1
            int index = --count;
            if (index == 0) {  // 线程都到达集合点
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    if (command != null)
                        // 如果barrierCommand不为null就执行
                        command.run();
                    ranAction = true;
                    nextGeneration();
                    return 0;
                } finally {
                    if (!ranAction)
                        breakBarrier();
                }
            }

            // loop until tripped, broken, interrupted, or timed out
            for (;;) {
                try {
                    if (!timed)
                        // 阻塞等待
                        trip.await();
                    else if (nanos > 0L)
                        // 超时的情况
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    // // 如果等待过程中，线程被中断
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        // We're about to finish waiting even if we had not
                        // been interrupted, so this interrupt is deemed to
                        // "belong" to subsequent execution.
                        Thread.currentThread().interrupt();
                    }
                }

                if (g.broken)
                    throw new BrokenBarrierException();
				// 如果“generation已经换代”，则返回index。
                if (g != generation)
                    return index;
				// 如果是“超时等待”，并且时间已到，则通过breakBarrier()终止CyclicBarrier，唤醒CyclicBarrier中所有等待线程，并抛出TimeoutException异常
                if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            lock.unlock();
        }
    }
```

**await(long timeout, TimeUnit unit)**

与上面的await方法功能基本一致，只不过这里有超时限制，阻塞等待直至到达超时时间为止

**getNumberWaiting()**

获取当前有多少个线程阻塞等待在临界点上

**isBroken()**

用于查询阻塞等待的线程是否被中断

**reset()**

将屏障重置为初始状态，如果当前有线程正在临界点等待的话，将抛出BrokenBarrierException

### 实现方式

基于ReentrantLock和Condition机制实现。除了getParties()方法，CyclicBarrier的其他方法都需要获取锁。 

CyclicBarrier的内部定义了一个Lock对象，每当一个线程调用`await()`时，将拦截的线程数减1，然后判断剩余拦截数是否为初始值parties，如果不是，进入Lock对象的条件队列等待。如果是，执行barrierAction对象的Runnable方法，然后将锁的条件队列中的所有线程放入锁等待队列中，这些线程会依次的获取锁、释放锁 。

对于失败的同步尝试，CyclicBarrier 使用了一种 all-or-none 的破坏模式：如果因为中断、失败或者超时等原因，导致线程过早地离开了屏障点，那么在该屏障点等待的其他所有线程也将通过 BrokenBarrierException（如果它们几乎同时被中断，则用 InterruptedException）以反常的方式离开。 

### 解除阻塞的情况

- 最后一个线程调用await()

- 当前线程被中断

- 其他正在该CyclicBarrier上等待的线程被中断

- 其他正在该CyclicBarrier上等待的线程超时

- 其他某个线程调用该CyclicBarrier的reset()方法

**使用CyclicBarrier的线程都会阻塞在await方法上，所以在线程池中使用CyclicBarrier时要特别小心，如果线程池的线程过少，那么就会发生死锁了**

## 区别

- CountDownLatch只能使用一次，CyclicBarrier可以使用多次
- CountDownLatch某线程运行到某个点上之后，只是给某个数值-1而已，该线程继续运行不会阻塞，CyclicBarrier某个线程运行到某个点上之后，该线程阻塞，直到所有的线程都到达了这个点，线程才继续执行
- CountDownLatch是线程组之间的等待，即一个或多个线程等待一组线程完成某件事情之后再执行；而CyclicBarrier则是线程组内的等待，即每个线程相互等待，即n个线程都被拦截之后，然后依次执行



Semaphore可以理解为**信号量**，用于**控制资源能够被并发访问的线程数量**。线程需要通过`acquire()`获取访问许可才能继续往下执行，否则只能在该方法处阻塞等待。当任务完成后，需要通过`release()`方法讲访问许可归还，以便其他线程能够继续获得访问许可，可以通过构造函数指定是否具有公平性，默认非公平性，这样也是为了保证吞吐量，不同的是它们获取信号量的机制：对于公平信号量而言，线程在尝试获取信号量许可时如果当前线程不在CLH队列的头部，则排队等候；而对于非公平信号量而言，无论当前线程是不是在CLH队列的头部，它都会直接获取信号量。

**Semaphore用来做本地特殊资源的并发访问控制是相当合适的，如果需要进行流量控制，优先使用Semaphore**