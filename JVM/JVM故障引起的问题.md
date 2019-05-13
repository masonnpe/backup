---
title: JVM故障引起的问题
categories: JVM
tags: [JVM]
---

- nio操作用到堆外内存，堆外内存只能等老年代满了以后fullgc 顺便回收一下，否则会一直等到oom
- 异步处理  等待的线程太多  积压了很多socket，jvm直接崩溃，所有改成生产者消费者的模式

<!--more-->