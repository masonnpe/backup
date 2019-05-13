一个大型服务不可避免的需要依赖其他服务，并且有可能需要通过网络请求依赖第三方客户端。这样就有可能因为单个依赖服务延迟而导致整个服务器上的资源被阻塞。更糟糕的是，倘若两个服务相互依赖，有一个服务对另一个服务响应延时就有可能造成雪崩效应，导致两个服务一起崩溃。

现如今微服务架构十分流行，其解决依赖隔离方案hystrix也被大家所认知。但目前还有很多服务还是停留在Spring mvc框架，无法直接使用Spring Cloud集成的hystrix方案。本文先简单介绍hystrix的基本知识，然后介绍hystrix在Spring mvc的使用，最后简单介绍下如何实现项目的hystrix信息监控。

# 一、简介

## 1、为什么要使用Hystrix

在复杂的分布式结构中，每个应用都可能会依赖很多其他的服务，并且这些服务都不可避免地有失效的可能。倘若没有对依赖失败进行隔离，那整个服务可能就会有被拖垮的风险。

例如，一个应用依赖了 30 个服务，并且每个服务能保证 99.99% 的可用率，下面是一些计算结果：

>
> 可用率：99.99%^30=99.7%
> 1亿次请求*0.3%=300,000次失效
> 换算成时间大约每个月2个小时服务不稳定。

然而，现实更加残酷，如果你没有针对整个系统做快速恢复，即使所有依赖只有 0.01% 的不可用率，累积起来每个月给系统带来的不可用时间也有数小时之多。

当所有依赖都正常，一个请求的拓扑结构如下所示：

[![img](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-1.png)](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-1.png)

当一个依赖服务有延迟，它将会阻塞整个用户请求：

[![img](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-2.png)](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-2.png)

在高QPS的环境下，一个依赖服务的延迟会导致整个服务器上资源都被阻塞。

应用中每一个网络请求或者间接通过客户端库发出的网络请求都是潜在的导致应用失效的原因。更严重的是，这些应用可能被其他服务依赖，由于每个服务都有诸如请求队列，线程池，或者其他系统资源等，一旦某个服务失效或者延迟增高，会导致更严重的级联失效。

[![img](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-3.png)](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-3.png)

hystrix被设计用来：

- 在通过第三方客户端访问（通常是通过网络）依赖服务出现高延迟或者失败时，为系统提供保护和控制
- 在分布式系统中防止级联失败
- 可以进行快速失败（不需要等待）和快速恢复（当依赖服务失效后又恢复正常，其对应的线程池会被清理干净，即剩下的都是未使用的线程，相对于整个 Tomcat 容器的线程池被占满需要耗费更长时间以恢复可用来说，此时系统可以快速恢复）
- 提供失败回退（Fallback）和优雅的服务降级机制
- 提供近实时的监控、报警和运维控制手段

## 2、Hystrix如何解决依赖隔离

- 将所有请求外部系统（或者叫依赖服务）的逻辑封装到 HystrixCommand 或者 HystrixObservableCommand 对象中，这些逻辑将会在独立的线程中被执行（利用了设计模式中的 Command模式）
- 对那些耗时超过设置的阈值的请求，Hystrix 采取自动超时的策略。该策略默认对所有 Command 都有效，当然，你也可以通过设置 Command 的配置以自定义超时时间，以使你的依赖服务在引入 Hystrix 之后能达到 99.5% 的性能
- 为每一个依赖服务维护一个线程池（或者信号量），当线程池占满，该依赖服务将会立即拒绝服务而不是排队等待
- 划分出成功、失败（抛出异常）、超时或者线程池占满四种请求依赖服务时可能出现的状态
- 引入『熔断器』机制，在依赖服务失效比例超过阈值时，手动或者自动地切断服务一段时间
- 当请求依赖服务时出现拒绝服务、超时或者短路（多个依赖服务顺序请求，前面的依赖服务请求失败，则后面的请求不会发出）时，执行该依赖服务的失败回退逻辑
- 近实时地提供监控和配置变更

当使用 Hystrix 包装了你的所有依赖服务的请求后，拓扑图如下
[![img](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-4.png)](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-4.png)

## 3、hystrix如何执行

hystrix执行分为三种模式，分别为同步执行、异步执行、Reactive模式执行。

1. 同步执行：若原方法返回参数非Future对象且非Observable对象则会构建该模式。使用command.execute()，阻塞，当依赖服务响应（或者抛出异常/超时）时，返回结果；
2. 异步执行：若原方法返回参数为Future对象时构建该模式。使用command.queue()，返回Future对象，通过该对象异步得到返回结果；
3. Reactive模式执行：若原方法返回参数为Observable对象时构建该模式。该模式又分observe()命令和toObservable()命令。observe()命令会立即发出请求，在依赖服务响应（或者抛出异常/超时）时，通过注册的 Subscriber得到返回结果。toObservable()命令只有在订阅该对象时，才会发出请求，然后在依赖服务响应（或者抛出异常/超时）时，通过注册的Subscriber得到返回结果。

在内部实现中，execute()是同步调用，内部会调用queue().get()方法。queue()内部会调用toObservable().toBlocking().toFuture()。也就是说，HystrixCommand 内部均通过一个Observable的实现来执行请求，即使这些命令本来是用来执行同步返回回应这样的简单逻辑。

[![img](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-5.png)](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-5.png)

1. 构建HystrixCommand或者HystrixObservableCommand对象；
2. 执行命令（execute()、queue()、observe()、toObservable()）；
3. 如果请求结果缓存这个特性被启用，并且缓存命中，则缓存的回应会立即通过一个Observable对象的形式返回；
4. 检查熔断器状态，确定请求线路是否是开路，如果请求线路是开路，Hystrix将不会执行这个命令，而是直接使用『失败回退逻辑』（即不会执行run()，直接执行getFallback()）；
5. 如果和当前需要执行的命令相关联的线程池和请求队列（或者信号量，如果不使用线程池）满了，Hystrix 将不会执行这个命令，而是直接使用『失败回退逻辑』（即不会执行run()，直接执行getFallback()）；
6. 执行HystrixCommand.run()或HystrixObservableCommand.construct()，如果这两个方法执行超时或者执行失败，则执行getFallback()；如果正常结束，Hystrix 在添加一些日志和监控数据采集之后，直接返回回应；
7. Hystrix 会将请求成功，失败，被拒绝或超时信息报告给熔断器，熔断器维护一些用于统计数据用的计数器。

这些计数器产生的统计数据使得熔断器在特定的时刻，能短路某个依赖服务的后续请求，直到恢复期结束，若恢复期结束根据统计数据熔断器判定线路仍然未恢复健康，熔断器会再次关闭线路。

## 4、hystrix基本配置

hystrix基本配置可以通过四种方式进行设置。

1. hystrix本身代码默认。这种是在以下三种都没有自定义的情况下使用，默认设置在hystrix-core下的HystrixCommandProperties和HystrixThreadPoolProperties
2. 自定义默认配置。可以使用配置文件进行全局默认配置。例如：hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds
3. 通过代码构造实例设置。
4. 动态实例配置。根据实例的key值（commandKey或者threadPollKey）通过配置文件给特定实例进行配置。例如，一个实例的commandKey为commandTest，则为hystrix.command.commandTest.execution.isolation.thread.timeoutInMilliseconds

本文只介绍hystrix常用的command和ThreadPool配置，其余配置可以查看[官网](https://github.com/Netflix/Hystrix/wiki/Configuration)

command配置

1. execution.isolation.strategy：执行隔离策略. Thread是默认推荐的选择。THREAD为每次在一个线程中执行，并发请求数限制于线程池的线程数。SEMAPHORE为在调用线程中执行，并发请求数限制于semaphore信号量的值。
2. execution.isolation.thread.timeoutInMilliseconds：超时时间，默认1000ms。
3. execution.timeout.enabled：是否开启超时，默认true。
4. execution.isolation.thread.interruptOnTimeout：当超时的时候是否中断(interrupt) HystrixCommand.run()执行，默认：true。
5. fallback.enabled：是否开启fallback，默认：true。
6. circuitBreaker.enabled：是否开启熔断，默认true。
7. circuitBreaker.requestVolumeThreshold：设置一个滑动窗口内触发熔断的最少请求量，默认20。例如，如果这个值是20，一个滑动窗口内只有19个请求时，即使19个请求都失败了也不会触发熔断。
8. circuitBreaker.sleepWindowInMilliseconds：设置触发熔断后，拒绝请求后多长时间开始尝试再次执行。默认5000ms。
9. circuitBreaker.errorThresholdPercentage：设置触发熔断的错误比例。默认50，即50%。
10. metrics.rollingStats.timeInMilliseconds：设置滑动窗口的统计时间。熔断器使用这个时间。默认10s
11. metrics.rollingStats.numBuckets：设置滑动统计的桶数量。默认10。metrics.rollingStats.timeInMilliseconds必须能被这个值整除。

threadPool配置

1. coreSize：设置线程池的core size,这是最大的并发执行数量。默认10。
2. maximumSize：设置线程池数量极大值，这是可以支持的最大并发量，一般情况下和coreSize是相等的。默认10。该值只有在allowMaximumSizeToDivergeFromCoreSize被设置时才能有效。
3. maxQueueSize：最大队列长度。设置BlockingQueue的最大长度。默认-1。 如果设置成-1，就会使用SynchronizeQueue。 如果其他正整数就会使用LinkedBlockingQueue。
4. queueSizeRejectionThreshold：设置拒绝请求的临界值。只有maxQueueSize为-1时才有效。设置设个值的原因是maxQueueSize值运行时不能改变，我们可以通过修改这个变量动态修改允许排队的长度。默认5。（注意：hystrix为每一个依赖服务维护一个线程池或者信号量，当线程池占满+queueSizeRejectionThreshold占满，该依赖服务将会立即拒绝服务而不是排队等待）
5. keepAliveTimeMinutes：设置keep-live时间。默认1分钟。当coreSize==maximumSize时线程池是固定的。只有allowMaximumSizeToDivergeFromCoreSize值设置为true，coreSize和maximumSize才能分成两个部分。当coreSize < maximumSize，该值控制一个线程多久没使用才被释放。
6. allowMaximumSizeToDivergeFromCoreSize：该值确认maximumSize是否起作用。默认false。
7. metrics.rollingStats.timeInMilliseconds：和command配置含义一样。
8. metrics.rollingStats.numBuckets：和command配置含义一样。

倘若使用配置文件进行配置，两种配置可以根据key的字符串进行区分。command都是hystrix.command.commandKey(or default).属性名，threadpool都是hystrix.threadpool.threadpoolKey(groupKey or default).属性名。

# 二、hystrix在spring mvc的使用

hystrix在Spring cloud的使用非常简单，网上也有很多文档，在此就不多讲了。

为使熔断控制和现有代码解耦，hystrix[官方](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-javanica)采用了Aspect方式。现在介绍hystrix在spring mvc的使用。

## 1、添加依赖

使用maven引入hystrix依赖：

```
<dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-javanica</artifactId>
    <version>1.5.12</version>
</dependency>
```

## 2、添加配置

新建hystrix.properties文件（名字随意定，里面将定义项目所有hystrix配置信息）

新建一个类HystrixConfig

```
public class HystrixConfig
{
    public void init()
    {
        Properties prop = new Properties();
        InputStream in = null;
        try
        {
            in = HystrixConfig.class.getClassLoader().getResourceAsStream("hystrix.properties");
            prop.load(in);
            in.close();
            System.setProperties(prop);
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```

在spring的配置文件添加内容：

```
<!-- 添加了就不用加了 -->
<aop:aspectj-autoproxy proxy-target-class="true" />
<bean name="hystrixCommandAspect" class="com.netflix.hystrix.contrib.javanica.aop.aspectj.HystrixCommandAspect"/>
<bean id="hystrixConfig"  class="包名.HystrixConfig" init-method="init"/>
```

新建hystrixConfig bean主要是因为使用spring自带的context:property-placeholder配置加载器，hystrix无法读取。目前我只想到了通过System.setProperties的方式，若有其他方式欢迎指导。

### 3、hystrixCommand使用

举个简单的例子(写成接口方式是方便测试，普通的方法效果是一样的)：

```
@ResponseBody
@RequestMapping("/test.html")
@HystrixCommand
public String test(int s)
{
    logger.info("test.html start,s:{}", s);
    try
    {
        Thread.sleep(s * 1000);
    }
    catch (Exception e)
    {
        logger.error("test.html error.", e);
    }
    return "OK";
}
```

根据例子，我们可以看到和其他方法相比就添加了个@HystrixCommand注解，方法执行后会被HystrixCommandAspect拦截，拦截后会根据方法的基本属性（所在类、方法名、返回类型等）和HystrixCommand属性生成HystrixInvokable，最后执行。例子中，因为HystrixCommand属性为空，所以其groupKey默认为类名，commandKey为方法名。

通过HystrixCommand源码来看下可以设置的属性：

```
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface HystrixCommand {

    String groupKey() default "";

    String commandKey() default "";

    String threadPoolKey() default "";

    String fallbackMethod() default "";

    HystrixProperty[] commandProperties() default {};

    HystrixProperty[] threadPoolProperties() default {};

    Class<? extends Throwable>[] ignoreExceptions() default {};

    ObservableExecutionMode observableExecutionMode() default ObservableExecutionMode.EAGER;

    HystrixException[] raiseHystrixExceptions() default {};

    String defaultFallback() default "";
}
```

其中比较重要的是groupKey、commandKey、fallbackMethod（Fallback时调用的方法，一定要在同一个类中，且传参和返参要一致）。threadPoolKey一般可以不定义，线程池名会默认定义为groupKey。

再来看下HystrixCommandAspect是如何实现拦截的：

```
@Pointcut("@annotation(com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand)")
public void hystrixCommandAnnotationPointcut() {
}

@Pointcut("@annotation(com.netflix.hystrix.contrib.javanica.annotation.HystrixCollapser)")
public void hystrixCollapserAnnotationPointcut() {
}

@Around("hystrixCommandAnnotationPointcut() || hystrixCollapserAnnotationPointcut()")
public Object methodsAnnotatedWithHystrixCommand(final ProceedingJoinPoint joinPoint) throws Throwable {
    Method method = getMethodFromTarget(joinPoint);//见步骤1
    Validate.notNull(method, "failed to get method from joinPoint: %s", joinPoint);
    if (method.isAnnotationPresent(HystrixCommand.class) && method.isAnnotationPresent(HystrixCollapser.class)) {
        throw new IllegalStateException("method cannot be annotated with HystrixCommand and HystrixCollapser " +
                "annotations at the same time");
    }
    MetaHolderFactory metaHolderFactory = META_HOLDER_FACTORY_MAP.get(HystrixPointcutType.of(method));//见步骤2
    MetaHolder metaHolder = metaHolderFactory.create(joinPoint);//见步骤3
    HystrixInvokable invokable = HystrixCommandFactory.getInstance().create(metaHolder);//见步骤4
    ExecutionType executionType = metaHolder.isCollapserAnnotationPresent() ?
            metaHolder.getCollapserExecutionType() : metaHolder.getExecutionType();

    Object result;
    try {
        if (!metaHolder.isObservable()) {
            result = CommandExecutor.execute(invokable, executionType, metaHolder);
        } else {
            result = executeObservable(invokable, executionType, metaHolder);//见步骤5
        }
    } catch (HystrixBadRequestException e) {
        throw e.getCause() != null ? e.getCause() : e;
    } catch (HystrixRuntimeException e) {
        throw hystrixRuntimeExceptionToThrowable(metaHolder, e);
    }
    return result;
}
```

- 步骤1：获取切入点方法；
- 步骤2：根据方法的注解HystrixCommand或者HystrixCollapser生成相应的CommandMetaHolderFactory或者CollapserMetaHolderFactory类。
- 步骤3：将原方法的属性set进metaHolder中；
- 步骤4：根据metaHolder生成相应的HystrixCommand，包含加载hystrix配置信息。commandProperties加载的优先级为前缀hystrix.command.commandKey > hystrix.command.default > defaultValue(原代码默认)；threadPool配置加载的优先级为 前缀hystrix.threadpool.groupKey.> hystrix.threadpool.default.> defaultValue(原代码默认).
- 步骤5：执行命令。

倘若需要给该方法指定groupKey和commandKey定义其fallback方法，则可通过添加注解属性来实现。如：

```
@ResponseBody
@RequestMapping("/test.html")
@HystrixCommand(groupKey = "groupTest", commandKey = "commandTest", fallbackMethod = "back")
public String test(int s)
{
    try
    {
        Thread.sleep(s * 1000);
    }
    catch (Exception e)
    {
    }
    logger.info("test.html start");
    return "OK";
}

private String back(int s)
{
    return "back";
}
```

- groupKey=”groupTest”是将该hystrix操作的组名定义为groupTest，该属性在读取threadPoolProperties时需要用到。读取的策略是先读取已groupTest为键值的配置缓存；若没有则读取已hystrix.threadpool.groupTest.为前缀的配置；若没有则读取hystrix.threadpool.为前缀的配置，最后才读取代码默认的值。
- commandKey=”commandTest”是将hystrix操作的命令名定义为commandTest，该属性在读取commandProperties时需要用到。读取的策略与上面的一致，只是前缀由hystrix.threadpool变为hystrix.command。
- fallbackMethod=”back”是给该hystrix操作定义一个回退方法，值为回退方法的方法名，并且要与回退方法在同一个类下、相同的参入参数和返回参数。fallbackMethod可级联。

如果要给该方法指定一些hystrix属性，可通过在hystrix.properties中添加一些配置来实现。如给上述方法添加一些hystrix属性，示例如下：

```
#定义commandKey为commandTest的过期时间为3s
hystrix.command.commandTest.execution.isolation.thread.timeoutInMilliseconds=3000
#定义所有的默认过期时间为5s，不再是默认是1s。优先级小于上面配置
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=5000
#定义threadPoolKey为groupTest的线程池大小为15
hystrix.threadpool.groupTest.coreSize=15
#定义所有的线程池大小为为5，不再是默认是10。优先级小于上面配置
hystrix.threadpool.default.coreSize=5
```

其余的配置方式与例子中的相似，就不一一列举了。

至此，spring mvc就可以为每一个依赖随心添加依赖隔离了。

# 三、监控hystrix

除了隔离依赖服务的调用外，Hystrix还提供了近乎实时的监控，Hystrix会实时的，累加的记录所有关于HystrixCommand的执行信息，包括执行了每秒执行了多少请求，多少成功，多少失败等等。更多指标可以查看[官网](https://github.com/Netflix/Hystrix/wiki/Metrics-and-Monitoring)。

## 1、添加监控

### 添加依赖

使用maven引入hystrix-metrics-event-stream依赖：

```
<dependency>  
     <groupId>com.netflix.hystrix</groupId>  
     <artifactId>hystrix-metrics-event-stream</artifactId>  
     <version>1.1.2</version>  
 </dependency>
```

### 修改web.xml

在web.xml中添加代码：

```
<servlet>  
    <description></description>  
    <display-name>HystrixMetricsStreamServlet</display-name>  
    <servlet-name>HystrixMetricsStreamServlet</servlet-name>  
    <servlet-class>com.netflix.hystrix.contrib.metrics.eventstream.HystrixMetricsStreamServlet</servlet-class>  
</servlet>  

<servlet-mapping>  
    <servlet-name>HystrixMetricsStreamServlet</servlet-name>  
    <url-pattern>/hystrix.stream</url-pattern>  
</servlet-mapping>  
```

### 查看效果

配置好后，重新启动应用，访问[http://ip:port/appname/hystrix.stream，系统会不断刷新以获取实时的数据。](http://ip:port/appname/hystrix.stream%EF%BC%8C%E7%B3%BB%E7%BB%9F%E4%BC%9A%E4%B8%8D%E6%96%AD%E5%88%B7%E6%96%B0%E4%BB%A5%E8%8E%B7%E5%8F%96%E5%AE%9E%E6%97%B6%E7%9A%84%E6%95%B0%E6%8D%AE%E3%80%82)

## 2、Dashboard

可以看出，单纯使用字符输出的方式可读性太差，运维人员很难从中就看出系统的当前状态，于是Netflix又开发了一个开源项目[dashboard](https://github.com/Netflix/Hystrix/wiki/Dashboard)来可视化这些数据，帮助运维人员更直观的了解系统的当前状态，Dashboard使用起来非常方便，其就是一个Web项目，你只需要把[hystrix-dashboard.war](http://search.maven.org/remotecontent?filepath=com/netflix/hystrix/hystrix-dashboard/1.5.12/hystrix-dashboard-1.5.12.war)包下载下来，放到一个Web容器（Tomcat,Jetty等）中即可。

启动容器，访问[http://ip:port/hystrix-dashboard/#,就可以看到如下的界面：](http://ip:port/hystrix-dashboard/#,%E5%B0%B1%E5%8F%AF%E4%BB%A5%E7%9C%8B%E5%88%B0%E5%A6%82%E4%B8%8B%E7%9A%84%E7%95%8C%E9%9D%A2%EF%BC%9A)

[![img](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-7.png)](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-7.png)

按照上述操作点击monitor Streams,就可以查看该服务的hystrix监控了。监控界面如下：

[![img](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-8.png)](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-8.png)

可以看出，Dashboard主要展示了两类信息，一是HystrixCommand的执行情况，即circuit部分；二是线程池的状态，包括线程池名，大小，当前活跃线程说，最大活跃线程数，排队队列大小等，即Thread Pools部分。

然而，在复杂的分布式环境中，需要监控的不是单一一个ip的服务，可能需要监控一个集群甚至几个集群，而每个集群又可能有多个服务器，并且要可以扩展。倘若使用这种方案，运维人员需要添加N多监控路径。为解决该问题，Netflix又提供了一个开源项目[Turbine](https://github.com/Netflix/Turbine/wiki)来提供把多个hystrix.stream的内容聚合为一个数据源供Dashboard展示。

## 3、Turbine

部署turbine操作：

1. 下载[turbine-web-1.0.0.war](https://github.com/downloads/Netflix/Turbine/turbine-web-1.0.0.war)，并将war放入web容器中；
2. 在容器下路径为turbine-web-1.0.0/WEB-INF/classes下新建config.properties文件；
3. 根据实际情况配置相应参数，相应配置可以参考[官网](https://github.com/Netflix/Turbine/wiki/Configuration-(1.x))：
4. 调用[http://ip:port/turbine-web/turbine.stream?cluster=${clusterConfigName}，查看是否有数据；](http://ip:port/turbine-web/turbine.stream?cluster=${clusterConfigName}%EF%BC%8C%E6%9F%A5%E7%9C%8B%E6%98%AF%E5%90%A6%E6%9C%89%E6%95%B0%E6%8D%AE%EF%BC%9B)
5. 打开[http://ip:port/hystrix-dashboard/#添加相应的turbine](http://ip:port/hystrix-dashboard/#%E6%B7%BB%E5%8A%A0%E7%9B%B8%E5%BA%94%E7%9A%84turbine) Stream。

配置详情主要包括三个方面：

1. cluster配置：turbine一般会针对每一个cluster进行commandKeyThreadpoolcommandGroupKey数据聚合，其key名称为turbine.aggregator.clusterConfig，值为服务名称以逗号隔开；
2. instances配置：每个服务对应的ip，其key名称为turbine.ConfigPropertyBasedDiscovery.${clusterConfigName}.instances，${clusterConfigName}为服务名，值为ip以逗号分隔；
3. instanceUrlSuffix：hystrix监控url后缀，，其key名称为turbine.instanceUrlSuffix.${clusterConfigName}，值为端口+路径，其路径一般为/hystrix.stream。

三者之间的关系是，先定义clusterConfigName，然后根据instances和instanceUrlSuffix拼接出相应url，多个instances会将其metrics统计在一起，然后在[http://turbineIP:turbinePORT/turbine-web/turbine.stream?cluster=${clusterConfigName}下进行展示。](http://turbineip:turbinePORT/turbine-web/turbine.stream?cluster=${clusterConfigName}%E4%B8%8B%E8%BF%9B%E8%A1%8C%E5%B1%95%E7%A4%BA%E3%80%82)

示例如下图：

[![img](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-9.png)](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-9.png)

Dashboard操作如图：

[![img](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-10.png)](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-10.png)

展示界面如图：

[![img](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-11.png)](http://tech.lede.com/2017/06/15/rd/server/hystrix/hystrix-11.png)

至此，hystrix在spring mvc的应用及其监控操作全部完成。

# 四、参考链接

<https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-javanica>

<https://github.com/Netflix/Hystrix/wiki>

<https://github.com/Netflix/Hystrix/wiki/Dashboard>

<https://github.com/Netflix/Turbine/wiki>

[https://github.com/Netflix/Turbine/wiki/Configuration-(1.x)](https://github.com/Netflix/Turbine/wiki/Configuration-(1.x))

<http://youdang.github.io/categories/%E7%BF%BB%E8%AF%91/>

<http://www.cnblogs.com/java-zhao/p/5521233.html>