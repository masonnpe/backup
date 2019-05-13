---
title: FutureTask
categories: 并发编程
tags: [FutureTask]

---

FutureTask 实现了 RunnableFuture 接口，RunnableFuture 接口继承 Runnable 和 Future 接口，这使得 FutureTask 既可以当做一个任务执行，也可以有返回值。

```java
public class FutureTask<V> implements RunnableFuture<V>
public interface RunnableFuture<V> extends Runnable, Future<V>
```

FutureTask 可用于异步获取执行结果或取消执行任务的场景。FutureTask提供了启动和取消异步任务，查询异步任务是否计算结束以及获取最终的异步任务的结果的一些常用的方法。通过`get()`方法来获取异步任务的结果，但是会阻塞当前线程直至异步任务执行结束。一旦任务执行结束，任务不能重新启动或取消，除非调用`runAndReset()`方法。

在FutureTask的源码中为其定义了这些状态：

```
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
```

**get方法**

当FutureTask处于未启动或已启动状态时，执行FutureTask.get()方法将导致调用线程阻塞

如果FutureTask处于已完成状态，调用FutureTask.get()方法将导致调用线程立即返回结果或者抛出异常

**cancel方法**

当FutureTask处于**未启动状态**时，执行cancel()方法将此任务永远不会执行

当FutureTask处于**已启动状态**时，执行cancel(true)方法将以中断线程的方式来阻止任务继续进行，如果执行cancel(false)将不会对正在执行任务的线程有任何影响

当**FutureTask**处于已完成状态时，执行cancel(...)方法将返回false

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    Callable<Integer> callable=new MyCallable();
    ExecutorService service = Executors.newCachedThreadPool();
    // Future
    Future<Integer> future = service.submit(callable);
    System.out.println(future.get());
    // FutureTask
    FutureTask<Integer> task=new FutureTask<>(callable);
    service.submit(task);
    System.out.println(task.get());
}
```

