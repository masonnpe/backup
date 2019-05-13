---
title: ReentrantLock
categories: 并发编程
tags: [锁]
---

ReentrantLock重入锁，**支持重入性，表示能够对共享资源能够重复加锁，即当前线程获取该锁再次获取不会被阻塞**



**ReenTrantLock独有的能力：**

1. ReenTrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。
2. ReenTrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。
3. ReenTrantLock提供了一种能够中断等待锁的线程的机制，通过lock.lockInterruptibly()来实现这个机制。





当使用非公平锁的时候，会立刻尝试配置状态，成功了就会插队执行，失败了就会和公平锁的机制一样，调用`acquire()`方法，以排他的方式来获取锁，成功了立刻返回，否则将线程加入队列，知道成功调用为止。

## 重入性的实现原理

要想支持重入性，就要解决两个问题：

- 在线程获取锁的时候，如果已经获取锁的线程是当前线程的话则直接再次获取成功；
- 由于锁会被获取n次，那么只有锁在被释放同样的n次之后，该锁才算是完全释放成功。

<!--more-->

通过重写AQS的几个protected方法来表达自己的同步语义。针对第一个问题，我们来看看ReentrantLock是怎样实现的，以非公平锁为例，判断当前线程能否获得锁为例，核心方法为nonfairTryAcquire：

```
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    //1. 如果该锁未被任何线程占有，该锁能被当前线程获取
	if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
	//2.若被占有，检查占有线程是否是当前线程
    else if (current == getExclusiveOwnerThread()) {
		// 3. 再次获取，计数加一
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

这段代码的逻辑也很简单，具体请看注释。为了支持重入性，在第二步增加了处理逻辑，如果该锁已经被线程所占有了，会继续检查占有线程是否为当前线程，如果是的话，同步状态加1返回true，表示可以再次获取成功。每次重新获取都会对同步状态进行加一的操作，那么释放的时候处理思路是怎样的了？（依然还是以非公平锁为例）核心方法为tryRelease：

```
protected final boolean tryRelease(int releases) {
	//1. 同步状态减1
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
		//2. 只有当同步状态为0时，锁成功被释放，返回true
        free = true;
        setExclusiveOwnerThread(null);
    }
	// 3. 锁未被完全释放，返回false
    setState(c);
    return free;
}
```

代码的逻辑请看注释，需要注意的是，重入锁的释放必须得等到同步状态为0时锁才算成功释放，否则锁仍未释放。如果锁被获取n次，释放了n-1次，该锁未完全释放返回false，只有被释放n次才算成功释放，返回true。到现在我们可以理清ReentrantLock重入性的实现了，也就是理解了同步语义的第一条。

## 公平锁与公平锁

ReentrantLock支持两种锁：**公平锁**和**非公平锁**。

- 公平锁：每次获取到锁为同步队列中的第一个节点，**保证请求资源时间上的绝对顺序**非公平锁有可能刚释放锁的线程下次继续获取该锁，则有可能导致其他线程永远无法获取到锁，**造成“饥饿”现象**

- 公平锁为了保证时间上的绝对顺序，需要频繁的上下文切换，而非公平锁会降低一定的上下文切换，降低性能开销。ReentrantLock默认非公平锁，为了减少一部分上下文切换，**保证了系统更大的吞吐量**

公平锁的处理逻辑是怎样的，核心方法为：

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
  }
}
```

唯一的不同在于增加了hasQueuedPredecessors的逻辑判断，方法名就可知道该方法用来判断当前节点在同步队列中是否有前驱节点的判断，如果有前驱节点说明有线程比当前线程更早的请求资源，根据公平性，当前线程请求资源失败。如果当前节点没有前驱节点的话，再才有做后面的逻辑判断的必要性。**公平锁每次都是从同步队列中的第一个节点获取到锁，而非公平性锁则不一定，有可能刚释放锁的线程能再次获取到锁**。

## 使用场景

### tryLock

1. 比如一个定时任务,第一次定时任务未完成,重复发起了第二次,直接返回flase; 
2. 用在界面交互时点击执行较长时间请求操作时，防止多次点击导致后台重复执行  

```java
public class LockTest
{
    private static final ReentrantLock LOCK=new ReentrantLock();

    public static void tryLockTest(){
        if(LOCK.tryLock()){// 如果已经被lock，则立即返回false不会等待
            try {
                // ... method body
                System.out.println(Thread.currentThread().getName()+"lock");
                Thread.sleep(20);
            }
            catch (InterruptedException e) {
                e.printStackTrace();
            }
            finally {
                System.out.println(Thread.currentThread().getName()+"unlock");
                LOCK.unlock();
            }
        }
    }
}
```

### lock

1. 同步操作 类似于synchronized 如果被其它资源锁定，会在此等待锁释放，达到暂停的效果
2. ReentrantLock存在公平锁与非公平锁  而且synchronized都是公平的

```java
    public static void lockTest(){
        LOCK.lock();// //如果被其它资源锁定，会在此等待锁释放，达到暂停的效果
        try {
            // ... method body
            System.out.println(Thread.currentThread().getName()+"lock");
            Thread.sleep(20);
        }
        catch (InterruptedException e) {
            e.printStackTrace();
        }
        finally {
            System.out.println(Thread.currentThread().getName()+"unlock");
            LOCK.unlock();
        }
    }
```

### tryLock(timeout, unit)

1. 如果发现该操作正在执行,等待一段时间，如果规定时间未得到锁,放弃。防止资源处理不当，线程队列溢出,出现死锁

```java
    public static void tryLockTimeTest(){
        try {
            if(LOCK.tryLock(5000, TimeUnit.SECONDS)){// 如果已经被lock，尝试等待5s，看是否可以获得锁，如果5s后仍然无法获得锁则返回false继续执行
                try {
                    // ... method body
                    System.out.println(Thread.currentThread().getName()+"lock");
                    Thread.sleep(6000);
                }
                finally {
                    System.out.println(Thread.currentThread().getName()+"unlock");
                    LOCK.unlock();
                }
            }
        }
        catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

### lockInterruptibly

1. 中断正在进行的操作立刻释放锁继续下一操作.比如 取消正在同步运行的操作，来防止不正常操作长时间占用造成的阻塞

```java
    public static void lockInterruptiblyTest(){
        try {
            LOCK.lockInterruptibly();
            // ... method body
            System.out.println(Thread.currentThread().getName()+"lock");
            Thread.sleep(10000);
        }
        catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            System.out.println(Thread.currentThread().getName()+"unlock");
            LOCK.unlock();
        }
    }
```

测试：

```java
public static void main(String[] args) {
        for (int i = 0; i < 1000; i++) {
            new Thread(()->LockTest.lockTest()).start();
        }
    }
```

