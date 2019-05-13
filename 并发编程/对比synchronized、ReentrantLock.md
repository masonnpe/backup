---
title: 对比synchronized、ReentrantLock
categories: 并发编程
tags: [ReentrantLock,synchronized]
---

ReentrantLock使用起来比较灵活，但是必须手动获取与释放锁，而synchronized不需要手动释放和开启锁
ReentrantLock只适用于代码块锁，而synchronized可用于修饰方法、代码块
ReentrantLock的优势体现在：
具备尝试非阻塞地获取锁的特性：当前线程尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取并持有锁
能被中断地获取锁的特性：与synchronized不同，获取到锁的线程能够响应中断，当获取到锁的线程被中断时，中断异常将会被抛出，同时锁会被释放
超时获取锁的特性：在指定的时间范围内获取锁；如果截止时间到了仍然无法获取锁，则返回
3 注意事项
在使用ReentrantLock类的时，一定要注意三点：
在fnally中释放锁，目的是保证在获取锁之后，最终能够被释放
不要将获取锁的过程写在try块内，因为如果在获取锁时发生了异常，异常抛出的同时，也会导致锁无故被释放。
ReentrantLock提供了一个newCondition的方法，以便用户在同一锁的情况下可以根据不同的情况执行等待或唤醒的动作

锁降级确实是会发生的，当JVM进入安全点（SafePoint）的时候，会检查是否有闲置的Monitor，然后试图进行降级。