针对造成服务雪崩的不同原因, 可以使用不同的应对策略:

1. 流量控制
2. 改进缓存模式
3. 服务自动扩容
4. 服务调用者降级服务

**流量控制** 的具体措施包括:

- 网关限流
- 用户交互限流
- 关闭重试

因为Nginx的高性能, 目前一线互联网公司大量采用Nginx+Lua的网关进行流量控制, 由此而来的OpenResty也越来越热门.

用户交互限流的具体措施有: 1. 采用加载动画,提高用户的忍耐等待时间. 2. 提交按钮添加强制等待时间机制.

**改进缓存模式** 的措施包括:

- 缓存预加载
- 同步改为异步刷新

**服务自动扩容** 的措施主要有:

- AWS的auto scaling

**服务调用者降级服务** 的措施包括:

- 资源隔离
- 对依赖服务进行分类
- 不可用服务的调用快速失败

资源隔离主要是对调用服务的线程池进行隔离.

我们根据具体业务,将依赖服务分为: 强依赖和若依赖. 强依赖服务不可用会导致当前业务中止,而弱依赖服务的不可用不会导致当前业务的中止.

不可用服务的调用快速失败一般通过 **超时机制**, **熔断器** 和熔断后的 **降级方法** 来实现.

## 简介

Hystrix 是一种高可用保障的一个框架。在分布式系统中，每个服务都有很多依赖服务，为了不让依赖服务的故障导致本地服务的资源耗尽而崩溃，可以使用Hystrix对服务间的调用进行控制，加入一些调用延迟或者依赖故障的容错机制。Hystrix通过将依赖服务进行资源隔离，防止故障蔓延导致整个系统崩溃，同时Hystrix还提供故障时的fallback降级机制，提升分布式系统的可用性和稳定性。

## 设计原则

- 对依赖服务调用时出现的调用延迟和调用失败进行控制和容错保护，对于超出我们设定阈值的服务调用，直接进行超时，不允许其耗费过长时间阻塞住。
- 阻止故障蔓延，阻止任何一个依赖服务耗尽所有的资源，比如tomcat中的所有线程资源，使用资源隔离技术，比如bulkhead（舱壁隔离技术），swimlane（泳道技术），circuit breaker（短路技术），来限制任何一个依赖服务的故障的影响。通过HystrixCommand或者HystrixObservableCommand来封装对外部依赖的访问请求，这个访问请求一般会运行在独立的线程中，资源隔离。为每一个依赖服务维护一个独立的线程池，或者是semaphore，当线程池已满时，直接拒绝对这个服务的调用
- 提供快速失败和快速恢复的支持
- 当一个服务调用出现失败，被拒绝，超时，短路等异常情况时，fallback优雅降级的支持，避免请求排队和积压，采用限流和fail fast来控制故障。如果对一个依赖服务的调用失败次数超过了一定的阈值，自动进行熔断，在一定时间内对该服务的调用直接降级，一段时间后再自动尝试恢复。
- 支持近实时的监控、报警以及运维操作，来提高故障发现的速度，通过近实时的属性和配置热修改功能，来提高故障处理和恢复的速度。对依赖服务的调用的成功次数，失败次数，拒绝次数，超时次数，进行统计

## 流程图

使用 Hystrix 来包装请求依赖服务时的流程：

![流程图](https://youdang.github.io/2016/02/05/translate-hystrix-wiki-how-it-works/flow-chart.png)

下图为Hystrix服务调用的内部逻辑: 
![图片描述](https://segmentfault.com/img/bVziap)

1. 构建Hystrix的Command对象, 调用执行方法.
2. Hystrix检查当前服务的熔断器开关是否开启, 若开启, 则执行降级服务getFallback方法.
3. 若熔断器开关关闭, 则Hystrix检查当前服务的线程池是否能接收新的请求, 若超过线程池已满, 则执行降级服务getFallback方法.
4. 若线程池接受请求, 则Hystrix开始执行服务调用具体逻辑run方法.
5. 若服务执行失败, 则执行降级服务getFallback方法, 并将执行结果上报Metrics更新服务健康状况.
6. 若服务执行超时, 则执行降级服务getFallback方法, 并将执行结果上报Metrics更新服务健康状况.
7. 若服务执行成功, 返回正常结果.
8. 若服务降级方法getFallback执行成功, 则返回降级结果.
9. 若服务降级方法getFallback执行失败, 则抛出异常.

## 执行命令

Hystrix 进行资源隔离，其实是提供了一个抽象叫做command。Hystrix 命令提供四种方式（`HystrixCommand` 支持所有四种方式，而 `HystrixObservableCommand`仅支持后两种方式）来执行包装的请求：

| 方法         | 调用   | 描述                                                         |
| ------------ | ------ | ------------------------------------------------------------ |
| execute      | 同步   | 阻塞，当依赖服务响应（或者抛出异常/超时）时，返回结果。实际调用queue().get()，同步返回 run() 的执行结果 |
| queue        | 异步   | 返回 Future 对象，通过该对象异步得到返回结果。实际调用toObservable().toBlocking().toFuture()将 Observable 转换成BlockingObservable ，toFuture()返回可获得run()抽象方法执行结果的 Future |
| observe      | 异步   | 返回 Observable 对象，立即发出请求，在依赖服务响应（或者抛出异常/超时）时，通过注册的 Subscriber 得到返回结果。调用 toObservable() 方法的基础上，向 Observable 注册 ReplaySubject 发起订阅 。ReplaySubject 会发射所有来自原始 Observable 的数据给观察者，无论它们是何时订阅的 |
| toObservable | 未调用 | 返回 Observable 对象，但只有在订阅该对象时，才会发出请求，然后在依赖服务响应（或者抛出异常/超时）时，通过注册的 Subscriber 得到返回结果 |

```java
public abstract class HystrixCommand<R> extends AbstractCommand<R> implements HystrixExecutable<R>, HystrixInvokableInfo<R>, HystrixObservable<R> {
    
    public R execute() {
        try {
            return queue().get();
        } catch (Exception e) {
            throw Exceptions.sneakyThrow(decomposeException(e));
        }
    }
    
    
    public Future<R> queue() {
        
        final Future<R> delegate = toObservable().toBlocking().toFuture();
    	
        final Future<R> f = new Future<R>() {       

            @Override
            public R get() throws InterruptedException, ExecutionException {
                return delegate.get();
            }
        	
        };
    }
    
}
```

```java
abstract class AbstractCommand<R> implements HystrixInvokableInfo<R>, HystrixObservable<R> {

	public Observable<R> observe() {
        // us a ReplaySubject to buffer the eagerly subscribed-to Observable
        ReplaySubject<R> subject = ReplaySubject.create();
        // eagerly kick off subscription
        final Subscription sourceSubscription = toObservable().subscribe(subject);
        // return the subject that can be subscribed to later while the execution has already started
        return subject.doOnUnsubscribe(new Action0() {
            @Override
            public void call() {
                sourceSubscription.unsubscribe();
            }
        });
    }
    
    public Observable<R> toObservable() {

        return Observable.defer(new Func0<Observable<R>>() {
            
        });
    }
}
```

## 执行结果缓存

如果请求结果缓存这个特性被启用，并且缓存命中，则缓存的回应会立即通过一个 `Observable` 对象的形式返回。

```java
public Observable<R> toObservable() {

        return Observable.defer(new Func0<Observable<R>>() {
            @Override
            public Observable<R> call() {
                 /* 只能执行一次 */
                if (!commandState.compareAndSet(CommandState.NOT_STARTED, CommandState.OBSERVABLE_CHAIN_CREATED)) {
                    IllegalStateException ex = new IllegalStateException("This instance can only be executed once. Please instantiate a new instance.");
                    //TODO make a new error type for this
                    throw new HystrixRuntimeException(FailureType.BAD_REQUEST_EXCEPTION, _cmd.getClass(), getLogMessagePrefix() + " command executed multiple times - this is not permitted.", ex, null);
                }

                commandStartTimestamp = System.currentTimeMillis();

                if (properties.requestLogEnabled().get()) {
                    // log this command execution regardless of what happened
                    if (currentRequestLog != null) {
                        currentRequestLog.addExecutedCommand(_cmd);
                    }
                }

                final boolean requestCacheEnabled = isRequestCachingEnabled();// 缓存开关
                final String cacheKey = getCacheKey();

                /* 优先从缓存中取如果请求结果缓存这个特性被启用，并且缓存命中，则缓存的回应会立即通过一个 Observable 对象的形式返回 */
                if (requestCacheEnabled) {
                    HystrixCommandResponseFromCache<R> fromCache = (HystrixCommandResponseFromCache<R>) requestCache.get(cacheKey);
                    if (fromCache != null) {
                        isResponseFromCache = true;
                        return handleRequestCacheHitAndEmitValues(fromCache, _cmd);
                    }
                }
				// 获取执行命令的 Observable 
                Observable<R> hystrixObservable =
                        Observable.defer(applyHystrixSemantics)
                                .map(wrapWithAllOnNextHooks);

                Observable<R> afterCache;

                // 当缓存特性开启，并且缓存未命中时，创建【订阅了执行命令的 Observable】的 HystrixCommandResponseFromCache
                if (requestCacheEnabled && cacheKey != null) {
                    // 创建 HystrixCommandResponseFromCache ，并添加到 requestCache 。哟，HystrixRequestCache#putIfAbsent(...) 方法，多个线程添加时，只有一个线程添加成功
                    HystrixCachedObservable<R> toCache = HystrixCachedObservable.from(hystrixObservable, _cmd);
                    HystrixCommandResponseFromCache<R> fromCache = (HystrixCommandResponseFromCache<R>) requestCache.putIfAbsent(cacheKey, toCache);
                    if (fromCache != null) {
                        // 调用 HystrixCommandResponseFromCache#unsubscribe() 方法，取消 HystrixCommandResponseFromCache 的订阅。这一步很关键，因为我们不希望缓存不存在时，多个线程去执行命令，最好有且只有一个线程执行命令。
                        toCache.unsubscribe();
                        isResponseFromCache = true;// 标记从缓存中结果
                        return handleRequestCacheHitAndEmitValues(fromCache, _cmd);
                    } else {
                        // 添加成功，调用 HystrixCommandResponseFromCachetoObservable() 方法，获得缓存 Observable
                        afterCache = toCache.toObservable();
                    }
                } else {
                    afterCache = hystrixObservable; // 当缓存特性未开启，使用执行命令 Observable 
                }

                return afterCache
                        .doOnTerminate(terminateCommandCleanup)     // perform cleanup once (either on normal terminal state (this line), or unsubscribe (next line))
                        .doOnUnsubscribe(unsubscribeCommandCleanup) // perform cleanup once
                        .doOnCompleted(fireOnCompletedHook);
            }
        });
    }
```

## 断路器HystrixCircuitBreaker

Hystrix会将每一个依赖服务的调用成功，失败，拒绝，超时，等事件，都会发送给circuit breaker断路器，断路器就会对调用成功/失败/拒绝/超时等事件的次数进行统计，断路器会根据这些统计次数来决定，是否要进行短路，如果打开了短路器，那么在一段时间内就会直接短路，然后如果在之后第一次检查发现调用成功了，就关闭断路器。

HystrixCircuitBreaker 有三种状态 ：

- `CLOSED` ：关闭
- `OPEN` ：打开
- `HALF_OPEN` ：半开

其中，断路器处于 `OPEN` 状态时，链路处于**非健康**状态，命令执行时，直接调用**回退**逻辑，跳过**正常**逻辑。

### 工作原理

1. 如果经过短路器的流量超过了一定的阈值比如在10s内，经过短路器的流量必须达到20个如果在10s内，经过短路器的流量才10个那就不会去判断要不要短路
2. 如果断路器统计到的异常调用的占比超过了一定的阈值就会开启短路
3. 然后断路器从close状态转换到open状态
4. 断路器打开的时候，所有经过该断路器的请求全部被短路，不调用后端服务，直接走fallback降级
5. 经过了一段时间之后，会half-open，让一条请求经过短路器，看能不能正常调用。如果调用成功了，那么就自动恢复，转到close状态

### 配置

- circuitBreaker.enabled  断路器是否允许工作，包括跟踪依赖服务调用的健康状况，以及对异常情况过多时是否允许触发短路，默认是true
- circuitBreaker.requestVolumeThreshold  设置一个，最少要有多少个请求时，才触发开启短路
- circuitBreaker.errorThresholdPercentage 设置异常请求量的百分比，当异常请求达到这个百分比时，就触发打开短路器，默认是50
- circuitBreaker.sleepWindowInMilliseconds 设置在短路之后，需要在多长时间内直接reject请求，然后在这段时间之后，进入half-open状态，尝试允许请求通过以及自动恢复，默认值是5000毫秒

```java
public interface HystrixCircuitBreaker {

    /**
     * Every {@link HystrixCommand} requests asks this if it is allowed to proceed or not.  It is idempotent and does
     * not modify any internal state, and takes into account the half-open logic which allows some requests through
     * after the circuit has been opened
     * 
     * @return boolean whether a request should be permitted
     */
    boolean allowRequest();

    /**
     * Whether the circuit is currently open (tripped).
     * 
     * @return boolean state of circuit breaker
     */
    boolean isOpen();

    /**
     * Invoked on successful executions from {@link HystrixCommand} as part of feedback mechanism when in a half-open state.
     */
    void markSuccess();

    /**
     * Invoked on unsuccessful executions from {@link HystrixCommand} as part of feedback mechanism when in a half-open state.
     */
    void markNonSuccess();
 
    /**
     * Invoked at start of command execution to attempt an execution.  This is non-idempotent - it may modify internal
     * state.
     */
    boolean attemptExecution();
}
```

------

![img](http://www.iocoder.cn/images/Hystrix/2018_11_08/01.png)

在 AbstractCommand 创建时，初始化 HystrixCircuitBreaker ，根据circuitBreakerEnabled判断是否开启，具体由HystrixCircuitBreaker.Factory创建



subscribeToStream()向 Hystrix Metrics 对请求量统计 Observable 的发起订阅

```java
private Subscription subscribeToStream() {
    /*
     * This stream will recalculate the OPEN/CLOSED status on every onNext from the health stream
     */
    return metrics.getHealthCountsStream()
            .observe()
            .subscribe(new Subscriber<HealthCounts>() {// 向 Hystrix Metrics 对请求量统计 Observable 的发起订阅
                @Override
                public void onCompleted() {

                }

                @Override
                public void onError(Throwable e) {

                }

                @Override
                public void onNext(HealthCounts hc) {
                    // 先判断请求数量
                    if (hc.getTotalRequests() < properties.circuitBreakerRequestVolumeThreshold().get()) {
                        // we are not past the minimum volume threshold for the stat window,
                        // so no change to circuit status.
                        // if it was CLOSED, it stays CLOSED
                        // if it was half-open, we need to wait for a successful command execution
                        // if it was open, we need to wait for sleep window to elapse
                    } else {
                        // 错误请求比例
                        if (hc.getErrorPercentage() < properties.circuitBreakerErrorThresholdPercentage().get()) {
                            //we are not past the minimum error threshold for the stat window,
                            // so no change to circuit status.
                            // if it was CLOSED, it stays CLOSED
                            // if it was half-open, we need to wait for a successful command execution
                            // if it was open, we need to wait for sleep window to elapse
                        } else {
                            // our failure rate is too high, we need to set the state to OPEN
                            // 断路器打开并设置打开时间
                            if (status.compareAndSet(Status.CLOSED, Status.OPEN)) {
                                circuitOpened.set(System.currentTimeMillis());
                            }
                        }
                    }
                }
            });
}
```

attemptExecution  判断能否回正常状态

```
private Observable<R> applyHystrixSemantics(final AbstractCommand<R> _cmd) {
    // mark that we're starting execution on the ExecutionHook
    // if this hook throws an exception, then a fast-fail occurs with no fallback.  No state is left inconsistent
    executionHook.onStart(_cmd);

    /* determine if we're allowed to execute */
    if (circuitBreaker.attemptExecution()) {
        // 正常
    } else {
    	// fallback
        return handleShortCircuitViaFallback();
    }
}
```

  

## 线程池/请求队列/信号量占满时会发生什么

如果和当前需要执行的命令相关联的线程池和请求队列（或者信号量，如果不使用线程池），Hystrix 将不会执行这个命令，而是直接使用失败回退逻辑

## 执行请求

Hystrix 将根据你使用类的不同，内部使用不同的方式来请求依赖服务：

- `HystrixCommand.run()` —— 返回回应或者抛出异常
- `HystrixObservableCommand.construct()` —— 返回 Observable 对象，并在回应到达时通知 observers，或者回调 `onError` 方法通知出现异常

若 `run()` 或者 `construct()` 方法耗时超过了给命令设置的超时阈值，执行请求的线程将抛出 `TimeoutException`（若命令本身并不在其调用线程内执行，则单独的定时器线程会抛出该异常）。在这种情况下，Hystrix 将会执行失败回退逻辑，并且会忽略最终（若执行命令的线程没有被中断）返回的回应。

若命令本身并不抛出异常，并正常返回回应，Hystrix 在添加一些日志和监控数据采集之后，直接返回回应。Hystrix 在使用 `run()` 方法时，Hystrix 内部还是会生成一个 `Observable` 对象，并返回单个请求，产生一个 `onCompleted` 通知；而在 Hystrix 使用 `construct()` 时，会直接返回由 `construct()` 产生的 `Observable` 对象

## 请求超时timeout

如果HystrixCommand.run()或HystrixObservableCommand.construct()的执行，超过了timeout时长的话，那么command所在的线程就会抛出一个TimeoutException，如果timeout了，也会去执行fallback降级机制，而且就不会管run()或construct()返回的值了。这里要注意的一点是，我们是不可能终止掉一个调用严重延迟的依赖服务的线程的，只能说给你抛出来一个TimeoutException，但是还是可能会因为严重延迟的调用线程占满整个线程池的。如果没有timeout的话，那么就会拿到一些调用依赖服务获取到的结果，然后hystrix会做一些logging记录和metric统计

## 失败回退Fallback

当命令执行失败时，Hystrix 将会执行失败回退逻辑，失败原因可能是：

1. `construct()` 或 `run()` 方法抛出异常
2. 当断路器open，导致命令被短路时
3. 当命令对应的线程池或信号量被占满
4. 超时

失败回退逻辑包含了通用的回应信息，这些回应从内存缓存中或者其他固定逻辑中得到，而不应有任何的网络依赖。两种最经典的降级机制：纯内存数据，默认值。如果一定要在失败回退逻辑中包含网络请求，必须将这些网络请求包装在另一个 `HystrixCommand` 或 `HystrixObservableCommand` 中。

当使用 `HystrixCommand` 时，通过实现 `HystrixCommand.getFallback()` 返回失败回退时的回应。

当使用 `HystrixObservableCommand` 时，通过实现 `HystrixObservableCommand.resumeWithFallback()` 返回 Observable 对象来通知 observers 失败回退时的回应。

若失败回退方法返回回应，Hystrix 会将这个回应返回给命令的调用者。若 Hystrix 内部调用 `HystrixCommand.getFallback()` 时，会产生一个 Observable 对象，并包装用户实现的 `getFallback()` 方法返回的回应；若 Hystrix 内部调用 `HystrixObservableCommand.resumeWithFallback()` 时，会将用户实现的 `resumeWithFallback()` 返回的 Observable 对象直接返回。

若你没有实现失败回退方法，或者失败回退方法抛出异常，Hystrix 内部还是会生成一个 Observable 对象，但它不会产生任何回应，并通过 `onError` 通知立即中止请求。Hystrix 默认会通过 `onError`通知调用者发生了何种异常。你需要尽量避免失败回退方法执行失败，保持该方法尽可能的简单不易出错。

若失败回退方法执行失败，或者用户未提供失败回退方法，Hystrix 会根据调用执行命令的方法的不同而产生不同的行为：

| 方法         | fallback为空或者异常时                                       |
| ------------ | ------------------------------------------------------------ |
| execute      | 直接抛出异常                                                 |
| queue        | 返回一个Future，调用get()时抛出异常                          |
| observe      | 返回一个Observable对象，但是调用subscribe()方法订阅它时，理解抛出调用者的onError方法 |
| toObservable | 返回一个Observable对象，但是调用subscribe()方法订阅它时，理解抛出调用者的onError方法 |

hystrix调用各种接口，或者访问外部依赖，mysql，redis，zookeeper，kafka，等等，如果出现了任何异常的情况

比如说报错了，访问mysql报错，redis报错，zookeeper报错，kafka报错，error

对每个外部依赖，无论是服务接口，中间件，资源隔离，对外部依赖只能用一定量的资源去访问，线程池/信号量，如果资源池已满，reject

访问外部依赖的时候，访问时间过长，可能就会导致超时，报一个TimeoutException异常，timeout

上述三种情况，都是我们说的异常情况，对外部依赖的东西访问的时候出现了异常，发送异常事件到短路器中去进行统计

如果短路器发现异常事件的占比达到了一定的比例，直接开启短路，circuit breaker

上述四种情况，都会去调用fallback降级机制

fallback，降级机制，你之前都是必须去调用外部的依赖接口，或者从mysql中去查询数据的，但是为了避免说可能外部依赖会有故障

比如，你可以再内存中维护一个ehcache，作为一个纯内存的基于LRU自动清理的缓存，数据也可以放入缓存内

如果说外部依赖有异常，fallback这里，直接尝试从ehcache中获取数据

比如说，本来你是从mysql，redis，或者其他任何地方去获取数据的，获取调用其他服务的接口的，结果人家故障了，人家挂了，fallback，可以返回一个默认值

## 返回正常回应

若命令成功被执行，Hystrix 将回应返回给调用方，或者通过 `Observable` 的形式返回。根据上述调用命令方式的不同（如第2条所示），`Observable` 对象会进行一些转换：

[![Observable对象的转化](https://youdang.github.io/2016/02/05/translate-hystrix-wiki-how-it-works/transformation-of-observable-object.png)](https://youdang.github.io/2016/02/05/translate-hystrix-wiki-how-it-works/transformation-of-observable-object.png)

- `execute()` —— 产生一个 `Future` 对象，行为同 `.queue()` 产生的 `Future` 对象一样，接着调用其 `get()` 方法，生成由内部产生的 `Observable` 对象返回的回应
- `queue()` —— 将内部产生的 `Observable` 对象转换（Decorator模式）成 `BlockingObservable` 对象，以产生并返回 `Future` 对象
- `observe()` —— 产生 `Observable` 对象后，立即订阅（ReplaySubject）以使命令得以执行（异步），返回该 `Observable` 对象，当你调用其 `subscribe` 方法时，重放产生的回应信息和通知给用户提供的订阅者
- `toObservable()` —— 返回 `Observable` 对象，你必须调用其 `subscribe` 方法，以使命令得以执行。

## 依赖隔离

Hystrix 通过使用bulkhead舱壁模式，将系统所有依赖服务隔离起来，并限制访问这些依赖服务的并发度。

### 线程池

适合绝大多数的场景，对依赖服务的网络请求的调用和访问，timeout这种问题

通过将对依赖服务的访问执行放到单独的线程，将其与调用线程（例如 Tomcat 线程池中的线程）隔离开来，调用线程能空出来去做其他的工作而不至于被依赖服务的访问阻塞过长时间。Hystrix 使用独立的，每个依赖服务对应一个线程池的方式，来隔离这些依赖服务，这样，某个依赖服务的高延迟只会拖慢这个依赖服务对应的线程池。

Netflix 在设计 Hystrix 时，使用线程池来实现隔离，原因如下：

- 多数系统同时运行了（有时甚至多达数百个）不同的后端服务，这些服务由不同开发组开发。
- 每个服务都提供了自己的客户端库
- 客户端库经常会发生变动
- 客户端库随时可能会增加新的网络请求的逻辑
- 客户端库可能会包含诸如自动重试，数据解析，内存中缓存等逻辑
- 客户端库一般都对调用者来说是个黑盒，包括实现细节，网络访问，默认配置，等等
- 即使客户端库本身未发生变化，依赖服务本身可能有会发生逻辑上的变化，也可能会影响其性能，从而导致客户端配置不再可靠
- 中间依赖服务可能包含一些其依赖服务提供的客户端库，而这些库可能不受控且配置不合理
- 大多数网络请求都是同步调用的
- 客户端代码可能也会有失效或者高延迟，而不仅仅是在网络访问时

#### 线程池的优势

将依赖服务请求通过使用不同的线程池隔离，其优势如下：

- 系统完全与依赖服务请求隔离开来，即使依赖服务对应线程池耗尽，也不会影响系统其它请求
- 降低了系统接入新的依赖服务的风险，若新的依赖服务存在问题，也不会影响系统其它请求
- 当依赖服务失效后又恢复正常，其对应的线程池会被清理干净，相对于整个 Tomcat 容器的线程池被占满需要耗费更长时间以恢复可用来说，此时系统可以快速恢复
- 若依赖服务的配置有问题，线程池能迅速反映出来（通过失败次数的增加，高延迟，超时，拒绝访问等等），同时，你可以在不影响系统现有功能的情况下，处理这些问题（通常通过热配置等方式）
- 若依赖服务的实现发生变更，性能有了很大的变化（这种情况时常发生），需要进行配置调整（例如增加/减小超时阈值，调整重试策略等）时，也可以从线程池的监控信息上迅速反映出来（失败次数增加，高延迟，超时，拒绝访问等等），同时，你可以在不影响其他依赖服务，系统请求和用户的情况下，处理这些问题
- 线程池处理能起到隔离的作用以外，还能通过这种内置的并发特性，在客户端库同步网络IO上，建立一个异步的 Facade（类似 Netflix API 建立在 Hystrix 命令上的 Reactive、全异步化的那一套 Java API）

简而言之，通过线程池提供的依赖服务隔离，可以使得我们能在不停止服务的情况下，更加优雅地应对客户端库和子系统性能上的变化。

注：尽管线程池能提供隔离性，但你仍然需要对你的依赖服务客户端代码增加超时逻辑，并且/或者处理线程中断异常，以使这些代码不会无故地阻塞或者拖慢 Hystrix 线程池。

#### 线程池的缺点

* 线程池机制最大的缺点就是增加了cpu的开销 除了tomcat本身的调用线程之外，还有Hystrix自己管理的线程池
* 每个command的执行都依托一个独立的线程，会进行排队，调度，还有上下文切换
* 用Hystrix带来额外的延迟

### 信号量

基于纯内存的一些业务逻辑服务，而不涉及到任何网络访问请求，对于这种访问系统内部的代码（不涉及任何的网络请求），只要做信号量的普通限流就可以了，避免内部复杂的低效率的代码，导致大量的线程被卡住。sempahore技术可以用来限流和削峰，但是不能用来对调研延迟的服务进行timeout和隔离

除了线程池，队列之外，你可以使用信号量（或者叫计数器）来限制单个依赖服务的并发度。Hystrix 可以利用信号量，而不是线程池，来控制系统负载，但信号量不允许我们设置超时和异步化，如果你对客户端库有足够的信任（延迟不会过高），并且你只需要控制系统负载，那么你可以使用信号量。

`HystrixCommand` 和 `HystrixObservableCommand` 在两个地方支持使用信号量：

- 失败回退逻辑：当 Hystrix 需要执行失败回退逻辑时，其在调用线程（Tomcat 线程）中使用信号量
- 执行命令时：如果设置了 Hystrix 命令的 `execution.isolation.strategy` 属性为 `SEMAPHORE`，则 Hystrix 会使用信号量而不是线程池来控制调用线程调用依赖服务的并发度

你可以通过动态配置（即热部署）来决定信号量的大小，以控制并发线程的数量，信号量大小的估计和使用线程池进行并发度估计一样（仅访问内存数据的请求，一般能达到耗时在 1ms 以内，且能达到 5000rps，这样的请求对应的信号量可以设置为 1 或者 2。默认值为 10）。

注意：如果依赖服务使用信号量来进行隔离，当依赖服务出现高延迟，其调用线程也会被阻塞，直到依赖服务的网络请求超时。

信号量在达到上限时，会拒绝后续请求的访问，同时，设置信号量的线程也无法异步化（即像线程池那样，实现『提交-做其他工作-得到结果』模式）

## 请求合并Collapser

你可以在 `HystrixCommand` 之前放置一个『请求合并器』（`HystrixCollapser` 为请求合并器的抽象父类），该合并器可以将多个发往同一个后端依赖服务的请求合并成一个。

### 为什么要使用请求合并？

在并发执行 `HystrixCommand` 时，利用请求合并能减少线程和网络连接数量。通过使用 `HystrixCollapser`，Hystrix 能自动完成请求的合并，开发者不需要对现有代码做批量化的开发。

##### 全局上下文（适用于所有 Tomcat 线程）

理想情况下，合并过程应该发生在系统全局层面，这样用户发起的，由 Tomcat 线程执行的所有请求都能被执行合并操作。

例如，有这样一个需求，用户需要获取电影评级，而这些数据需要系统请求依赖服务来获取，对依赖服务的请求使用 `HystrixCommand` 进行包装，并增加了请求合并的配置，这样，当同一个 JVM 中其他线程需要执行同样的请求时，Hystrix 会将这个请求同其他同样的请求合并，只产生一个网络请求。

注意：合并器会传递一个 `HystrixRequestContext` 对象到合并的网络请求中，因此，下游系统需要支持批量化，以使请求合并发挥其高效的特点。

##### 用户请求上下文（适用于单个 Tomcat 线程）

如果给 `HystrixCommand` 只配置成针对单个用户进行请求合并，则 Hystrix 只会在单个 Tomcat 线程（即请求）中进行请求合并。

例如，如果用户想加载 300 个视频对象的书签，请求合并后，Hystrix 会将原本需要发起的 300 个网络请求合并到一个。

##### 对象模型和代码复杂度

很多时候，当你创建一个对象模型，适用于对象的消费者逻辑，结果发现这个模型会导致生产者无法充分利用其拥有的资源。

例如，这里有一个包含 300 个视频对象的列表，需要遍历这个列表，并对每一个对象调用 `getSomeAttribute()` 方法，这是一个显而易见的对象模型，但如果简单处理的话，可能会导致 300 次的网络请求（假设 `getSomeAttribute()` 方法内需要发出网络请求），每一个网络请求可能都会花上几毫秒（显然，这种方式非常容易拖慢系统）。

当然，你也可以要求用户在调用 `getSomeAttribute()` 之前，先判断一下哪些视频对象真正需要请求其属性。

或者，你可以将对象模型进行拆分，从一个地方获取视频列表，然后从另一个地方获取视频的属性。

但这些实现会导致 API 非常丑陋，且实现的对象模型无法完全满足用户使用模式。 并且在企业级开发时，很容易因为开发者的疏忽导致错误或者不够高效，因为不同的开发者可能有不同的请求方式，这样一个地方的优化不足以保证在所有地方都会有优化。

通过将合并逻辑下沉到 Hystrix 层，不管你如何设计对象模型，或者以何种方式去调用依赖服务，又或者开发者是否意识到这些逻辑需要不需要进行优化，这些都不需要考虑，因为 Hystrix 能统一处理。

`getSomeAttribute()` 方法能放在它最适合的位置，并且能以最适合的方式被调用，Hystrix 的请求合并器会自动将请求合并到合并时间窗口内。

##### 请求合并带来的额外开销

请求合并会导致依赖服务的请求延迟增高（该延迟为等待请求的延迟），延迟的最大值为合并时间窗口大小。

若某个请求耗时的中位数是 5ms，合并时间窗口为 10ms，那么在最坏情况下（注：合并时间窗口开启时发起请求），请求需要消耗 15ms 才能完成。通常情况下，请求不太可能恰好在合并时间窗口开启时发起，因此，请求合并带来的额外开销应该是合并时间窗口的一般，在此例中是 5ms。

请求合并带来的额外开销是否值得，取决于将要执行的命令，高延迟的命令相比较而言不会有太大的影响。同时，缓存 Key 的选择也决定了在一个合并时间窗口内能『并发』执行的命令数量：如果一个合并时间窗口内只有 1~2 个请求，将请求合并显然不是明智的选择。事实上，如果单线程循环调用同一个依赖服务的情况下，如果将请求合并，会导致这个循环成为系统性能的瓶颈，因为每一个请求都需要等待 10ms 的合并时间周期。

然而，如果一个命令具有高并发度，并且能批量处理多个，甚至上百个的话，请求合并带来的性能开销会因为吞吐量的极大提升而基本可以忽略，因为 Hystrix 会减少这些请求所需的线程和网络连接数量。

##### 请求合并器的执行流程

[![请求合并器的执行流程](https://youdang.github.io/2016/02/05/translate-hystrix-wiki-how-it-works/collapser-flow.png)](https://youdang.github.io/2016/02/05/translate-hystrix-wiki-how-it-works/collapser-flow.png)

- 第一步，提交单个命令请求到请求队列( RequestQueue )
- 第二步，定时任务( TimerTask ) 固定周期从请求队列获取多个命令执行，合并执行。

## 请求缓存

在 `HystrixCommand` 和 `HystrixObservableCommand` 的实现中，可以定义一个缓存的 Key，这个 Key 用于在同一个请求上下文（全局或者用户级）中标识缓存的请求结果，当然，该缓存是线程安全的。

下例展示了在一个完整 HTTP 请求周期内，两个线程执行命令的流程：

[![请求缓存例子](https://youdang.github.io/2016/02/05/translate-hystrix-wiki-how-it-works/request-caching.png)](https://youdang.github.io/2016/02/05/translate-hystrix-wiki-how-it-works/request-caching.png)

请求缓存有如下好处：

- 不同请求路径上针对同一个依赖服务进行的重复请求（有同一个缓存 Key），不会真实请求多次

  这个特性在企业级系统中非常有用，在这些系统中，开发者往往开发的只是系统功能的一部分。（注：这样，开发者彼此隔离，不太可能使用同样的方法或者策略去请求同一个依赖服务提供的资源）

  例如，请求一个用户的 `Account` 的逻辑如下所示，这个逻辑往往在系统不同地方被用到：

  ```
  Account account = new UserGetAccount(accountId).execute();
  
  //or
  
  Observable<Account> accountObservable = new UserGetAccount(accountId).observe();
  ```

  Hystrix 的 `RequestCache` 只会在内部执行 `run()` 方法一次，上面两个线程在执行 `HystrixCommand` 命令时，会得到相同的结果，即使这两个命令是两个不同的实例。

- 数据获取具有一致性

  因为缓存的存在，除了第一次请求需要真正访问依赖服务以外，后续请求全部从缓存中获取，可以保证在同一个用户请求内，不会出现依赖服务返回不同的回应的情况。

- 避免不必要的线程执行

  在 `construct()` 或 `run()` 方法执行之前，会先从请求缓存中获取数据，因此，Hystrix 能利用这个特性避免不必要的线程执行，减小系统开销。

  若 Hystrix 没有实现请求缓存，那么 `HystrixCommand` 和 `HystrixObservableCommand` 的实现者需要自己在 `construct()` 或 `run()` 方法中实现缓存，这种方式无法避免不必要的线程执行开销。

在 toObservable() 方法里，如果请求结果缓存这个特性被启用，并且缓存命中，则缓存的回应会立即通过一个 Observable 对象的形式返回；如果缓存未命中，则返回的 ReplySubject 对象缓存执行结果（ReplySubject 能够重放执行结果，从而实现缓存的功效）。

**HystrixCachedObservable**

```
public class HystrixCachedObservable<R> {
    protected final Subscription originalSubscription;
    protected final Observable<R> cachedObservable;
    private volatile int outstandingSubscriptions = 0;

    protected HystrixCachedObservable(final Observable<R> originalObservable) {
        ReplaySubject<R> replaySubject = ReplaySubject.create();
        this.originalSubscription = originalObservable
                .subscribe(replaySubject);

        this.cachedObservable = replaySubject
                .doOnUnsubscribe(new Action0() {
                    @Override
                    public void call() {
                        outstandingSubscriptions--;
                        if (outstandingSubscriptions == 0) {
                            originalSubscription.unsubscribe();
                        }
                    }
                })
                .doOnSubscribe(new Action0() {
                    @Override
                    public void call() {
                        outstandingSubscriptions++;
                    }
                });
    }

    public static <R> HystrixCachedObservable<R> from(Observable<R> o, AbstractCommand<R> originalCommand) {
        return new HystrixCommandResponseFromCache<R>(o, originalCommand);
    }

    public static <R> HystrixCachedObservable<R> from(Observable<R> o) {
        return new HystrixCachedObservable<R>(o);
    }

    public Observable<R> toObservable() {
        return cachedObservable;
    }

    public void unsubscribe() {
        originalSubscription.unsubscribe();
    }
}

```

HystrixCachedObservable 不是一个 Observable 的子类，而是对传入的 Observable **封装** ：使用 ReplaySubject 向传入的 Observable 发起订阅，通过 ReplaySubject 能够**重放**执行结果，从而实现缓存的功效。传入的 `originalObservable`为 `hystrixObservable` 执行命令 Observable 。在 Hystrix 里，提供了两种执行命令的隔离方式 ：线程池 和信号量。

- 当使用 `THREAD` 隔离时，`#subscribe(replaySubject)` 调用完成时，**实际命令并未开始执行**，或者说，这是一个**异步**的执行命令的过程。那么，**会不会影响返回执行结果呢**？答案当然是不会，BlockingObservable 在得到执行完成才会**结束阻塞**，此时已经有执行结果。
- 当使用 `SEMAPHORE` 隔离时，`#subscribe(replaySubject)` 调用完成时，**实际命令已经执行完成**，所以即使 `AbstractCommand#toObservavle(...)` 调用 `HystrixCommandResponseFromCache#unsubscribe()` 方法，也会浪费，**重复**执行命令。而对于 `THREAD` 隔离的情况，通过取消订阅的方式，只会执行**一次**命令。当然，如果“恶搞” `THREAD` 隔离的情况，增加 `sleep` 的调用如下，就能达到**重复**执行命令的效果。

**HystrixCommandResponseFromCache** 

`com.netflix.hystrix.HystrixCommandResponseFromCache` ，是 HystrixCachedObservable 的子类。在父类的基础上，增加了对 `AbstractCommand.executionResult` 的关注。

`HystrixCachedObservable#from(Observable, AbstractCommand)` 方法，创建 HystrixCommandResponseFromCache 对象

`HystrixCommandResponseFromCache#toObservableWithStateCopiedInto(...)` 方法

- 通过completionLogicRun属性，保证#doOnError()#doOnCompleted()#doOnUnsubscribe()方法有且只有一个方法执行具体逻辑。
  - `#doOnError()` ，`#doOnCompleted()` 执行时，调用 `#commandCompleted()` 方法，从缓存命令( `HystrixCommandResponseFromCache.originalCommand` ) 复制 `executionResult` 属性给当前命令( `commandToCopyStateInto` ) 。
  - `#doOnUnsubscribe()` 执行时，调用 `#commandUnsubscribed()` 方法，使用当前命令( `commandToCopyStateInto`)**自己**的 `executionResult` ，不进行复制。

## 留着学

* request collapser，请求合并技术
* fail-fast和fail-slient，高阶容错模式
* static fallback和stubbed fallback，高阶降级模式
* 嵌套command实现的发送网络请求的降级模式
* 基于facade command的多级降级模式
* request cache的手动清理
* 生产环境中的线程池大小以及timeout配置优化经验
* 线程池的自动化动态扩容与缩容技术
* hystrix的metric高阶配置
* 基于hystrix dashboard的可视化分布式系统监控