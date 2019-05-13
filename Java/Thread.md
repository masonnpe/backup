---
title: 多线程
categories: 并发编程
tags: [Thread]
---

多线程的优势在于可以充分发挥多核CPU的计算能力，方便进行业务拆分，提高性能。缺点在于CPU是通过给线程分配时间片实现线程的切换，失去时间片时，线程需要记录当前执行的位置，等到分配到时间片时再恢复到之前的状态，线程上下文切换非常耗费性能，这也导致了公平锁和非公平锁之间的性能差距；还会导致线程安全问题，如死锁、读过期数据。

## 线程的状态

**NEW** ：线程创建后进入NEW状态

**RUNNABLE** ：线程创建后调用`start()`方法进入可运行态，分配到时间片后进入运行态

**WAITING** ：调用`wait()`、`join()`、`LockSupport.lock()`后线程进入WAITING状态

**TIMED_WAITING** ：调用`sleep(millis)`、`join(millis)`、`LockSupport.parkNanos(nanos)`、`LockSupport.parkUntil(deadline)`、`wait(timeout)`后线程进入TIMED_WAITING状态，比如java.util.concurrent.locks的lock，线程切换到的是WAITING状态或TIMED_WAITING状态，lock调用LockSupport的方法

**BLOCKED** ：使用synchronized线程出现锁竞争，竞争失败的线程会进入BLOCKED状态

**TERMINATED** ：线程正常运行结束或线程抛出一个未被捕捉的Exception或Error后会释放对象的监视器，线程进入TERMINATED状态

## 常用方法

### join()

可以用于线程的顺序执行，threadA调用threadB的`join()`方法后，线程A会一直阻塞，直到线程B终止后隐式调用`notifyAll()`方法唤醒所有等待线程后，threadA才会继续执行

```java
while (isAlive()) {
	wait(0);
}
```

### sleep()与wait()

* `sleep()`是Thread类的静态方法，`wait()`是Object类的实例方法
* `wait()`必须要在同步方法或同步块内才能调用，前提是已经获取对象锁，`sleep()`没有限制
* `wait()`会释放已占有的对象锁，`sleep()`不会释放
* `sleep()`在休眠指定时间后分配到时间片后就会继续执行，`wait()`必须等待`notify()`或`notifyAll()`通知后，才会离开等待队列，分配到时间片后才会继续执行
* 两者都可以暂停线程的执行。
* Wait通常被用于线程间交互/通信，sleep通常被用于暂停执行。

### yield()与sleep()

* 调用`yield()`后，当前线程会让出CPU，回到可运行状态等待时间片的分配，让出的时间片只会分配给与当前线程相同优先级或者更高优先级的线程；调用`sleep()`后，当前线程会让出CPU，其他线程都可以去竞争

### stop()、suspend()、resume()

`suspend()`线程挂起，不会释放锁

`resume()`线程继续执行，如果先于`suspend()`将会冻结

### wait()、notify()、notifyAll()

JDK强制wait()、notify()、notifyAll()方法在调用前都必须先获得对象的锁

wait()方法立即释放对象监视器，notify()/notifyAll()方法则会等待线程剩余代码执行完毕才会放弃对象监视器

### interrupt()、interrupted()、isInterrupted()

线程是因为调用了`wait()`、`sleep()`或者`join()`方法而导致的阻塞，可以中断线程，通过抛出InterruptedException来唤醒它

`interrupt()`对当前线程进行中断操作，如果线程调用了`wait()`、`join()`方法时会抛出InterruptedException并将中断标志位清除

`interrupted()`测试线程是否被中断，会清除中断标志位**当抛出InterruptedException时候，会清除中断标志位，也就是说在调用isInterrupted()会返回false**

**结束线程时通过中断标志位或者标志位的方式可以有机会去清理资源，相对于武断而直接的结束线程，这种方式要优雅和安全**

`isInterrupted()`测试线程是否被中断，不会清除中断标志位



void interrupt() 向线程发送中断请求，线程的中断状态将被设置为true，如果线程被一个sleep调用阻塞，那么将会抛出异常

static boolean interrupted() 测试当前线程是否被中断，他会将当前线程的中断状态重置为false

boolean isInterrupted 测试线程是否被终止   不改变线程的中断状态

## 常见问题

1. `Thread.holdsLock(obj)` 检测一个线程是否持有对象监视器锁

2. 通过`setDaemon(true)`可以将线程设置为守护线程，需要先于`start()`执行，否则会抛出一个异常，但线程还是会执行，只不过还是当作普通线程执行，当Java应用只有守护线程时，虚拟机就会自然退出，这时守护线程退出的时候并不会执行finally里的代码，所以将释放资源等操作放在finally块里时不安全的

**避免死锁**

- 避免一个线程同时获得多个锁，减少每个锁占用的资源数
- 尝试使用定时锁tryLock(timeout)，超过等待时间不会阻塞
- 数据库的加锁、解锁必须放在同一个数据库连接里





Java多线程中的死锁
死锁是指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。这是一个严重的问题，因为死锁会让你的程序挂起无法完成任务，死锁的发生必须满足以下四个条件：

- 互斥条件：一个资源每次只能被一个进程使用。
- 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
- 不剥夺条件：进程已获得的资源，在末使用完之前，不能强行剥夺。
- 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。

避免死锁最简单的方法就是阻止循环等待条件，将系统中所有的资源设置标志位、排序，规定所有的进程申请资源必须以一定的顺序（升序或降序）做操作来避免死锁。[这篇教程](http://javarevisited.blogspot.com/2010/10/what-is-deadlock-in-java-how-to-fix-it.html)有代码示例和避免死锁的讨论细节。

产生死锁的四个必要条件：

1.互斥（Mutual exclusion）：存在这样一种资源，它在某个时刻只能被分配给一个执行绪（也称为线程）使用；

2.持有（Hold and wait）：当请求的资源已被占用从而导致执行绪阻塞时，资源占用者不但无需释放该资源，而且还可以继续请求更多资源；

3.不可剥夺（No preemption）：执行绪获得到的互斥资源不可被强行剥夺，换句话说，只有资源占用者自己才能释放资源；

4.环形等待（Circular wait）：若干执行绪以不同的次序获取互斥资源，从而形成环形等待的局面，想象在由多个执行绪组成的环形链中，每个执行绪都在等待下一个执行绪释放它持有的资源。







一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

## InterruptedException

通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

对于以下代码，在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

```java
public class InterruptExample {

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println("Thread run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```java
public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new MyThread1();
    thread1.start();
    thread1.interrupt();
    System.out.println("Main run");
}
```

```html
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at InterruptExample.lambda$main$0(InterruptExample.java:5)
    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```

## interrupted()

如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

```java
public class InterruptExample {

    private static class MyThread2 extends Thread {
        @Override
        public void run() {
            while (!interrupted()) {
                // ..
            }
            System.out.println("Thread end");
        }
    }
}
```

```java
public static void main(String[] args) throws InterruptedException {
    Thread thread2 = new MyThread2();
    thread2.start();
    thread2.interrupt();
}
```

```html
Thread end
```

#### 20) Java中interrupted 和 isInterruptedd方法的区别？

*interrupted()* 和 *isInterrupted()*的主要区别是前者会将中断状态清除而后者不会。Java多线程的中断机制是用内部标识来实现的，调用*Thread.interrupt()*来中断一个线程就会设置中断标识为true。当中断线程调用[静态方法](http://java67.blogspot.com/2012/11/what-is-static-class-variable-method.html)*Thread.interrupted()*来检查中断状态时，中断状态会被清零。而非静态方法isInterrupted()用来查询其它线程的中断状态且不会改变中断状态标识。简单的说就是任何抛出InterruptedException异常的方法都会将中断状态清零。无论如何，一个线程的中断状态有有可能被其它线程调用中断来改变。

#### 37）如果你提交任务时，线程池队列已满。会时发会生什么？

这个问题问得很狡猾，许多程序员会认为该任务会阻塞直到线程池队列有空位。事实上如果一个任务不能被调度执行那么ThreadPoolExecutor’s submit()方法将会抛出一个RejectedExecutionException异常。
