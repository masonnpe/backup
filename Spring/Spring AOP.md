### AOP：

[※探秘Spring AOP（慕课网视频，很不错）](https://www.imooc.com/learn/869)

慕课网视频，讲解的很不错，详细且深入

[spring源码剖析（六）AOP实现原理剖析](https://blog.csdn.net/fighterandknight/article/details/51209822)

通过源码分析Spring AOP的原理

- 说说 Spring AOP
  - https://www.cnblogs.com/hongwz/p/5764917.html
- Spring AOP 实现原理
  - https://blog.csdn.net/moreevan/article/details/11977115/

Aspect-OrientedProgramming（面向切面编程）是对OOP的补充，目的是将那些与业务无关，却为业务模块所共同调用的逻辑封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。权限控制，错误处理，记录跟踪，事务等功能都会用到。

## 名词解释

切面（Aspect）：其实就是共有功能的实现。如日志切面、权限切面、事务切面等。在实际应用中通常是一个存放共有功能实现的普通Java类，之所以能被AOP容器识别成切面，是在配置中指定的。可以用Spring的 Advisor或拦截器实现。

通知（Advice）：是切面的具体实现。以目标方法为参照点，根据放置的地方不同，可分为前置通知（Before）、后置通知（AfterReturning）、异常通知（AfterThrowing）、最终通知（After）与环绕通知（Around）5种。在实际应用中通常是切面类中的一个方法，具体属于哪类通知，同样是在配置中指定的。

连接点（Joinpoint）：就是程序在运行过程中能够插入切面的地点。例如，方法调用、异常抛出或字段修改等，但Spring只支持方法级的连接点。

切入点（Pointcut）：用于定义通知应该切入到哪些连接点上。不同的通知通常需要切入到不同的连接点上，这种精准的匹配是由切入点的正则表达式来定义的。AOP框架必须允许开发者指定切入点：例如，使用正则表达式。 Spring定义了Pointcut接口，用来组合MethodMatcher和ClassFilter，可以通过名字很清楚的理解， MethodMatcher是用来检查目标类的方法是否可以被应用此通知，而ClassFilter是用来检查Pointcut是否应该应用到目标类上

目标对象（Target）：包含连接点的对象。也被称作被通知或被代理对象。就是那些即将切入切面的对象，也就是那些被通知的对象。这些对象中已经只剩下干干净净的核心业务逻辑代码了，所有的共有功能代码等待AOP容器的切入。

代理对象（Proxy）：将通知应用到目标对象之后被动态创建的对象。可以简单地理解为，代理对象的功能等于目标对象的核心业务逻辑功能加上共有功能。代理对象对于使用者而言是透明的，是程序运行过程中的产物。AOP框架创建的对象，包含通知。 在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理。

织入（Weaving）：将切面应用到目标对象从而创建一个新的代理对象的过程。这个过程可以发生在编译期、类装载期及运行期，当然不同的发生点有着不同的前提条件。譬如发生在编译期的话，就要求有一个支持这种AOP实现的特殊编译器（例如使用AspectJ编译器）；发生在类装载期，就要求有一个支持AOP实现的特殊类装载器；只有发生在运行期，则可直接通过Java语言的反射机制与动态代理机制来动态实现。

## AOP实现

实现AOP的技术，主要分为两大类：

- 静态AOP实现：AspectJ是一个基于Java语言的AOP框架，提供了强大的AOP功能，在编译阶段对程序进行修改，即实现对目标类的增强，生成静态AOP代理类
- 动态AOP实现：采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行。SpringAOP中如果目标对象实现了接口，Spring默认情况下会采用JDK的动态代理实现AOP，如果目标对象没有实现接口，必须采用CGLIB库。

### AspectJ

```java
@Component
@Aspect
public class MyAspect {
    private static final Logger LOGGER= LoggerFactory.getLogger(MyAspect.class);

    private static ThreadLocal<Date> threadLocal=new ThreadLocal<>();

    @Autowired(required=false)
    private HttpServletRequest request;

    @Pointcut("execution(* *.print())")
    public void exec(){}

    @Before(value = "exec()")
    public void beforeLog(){
        threadLocal.set(new Date());
        String requestURI = request.getRequestURI();
        String ipAddr = getIpAddr(request);
        LOGGER.info("type:{},uri:{},ipaddr:{}","before",requestURI,ipAddr);
    }

    @After(value = "exec()")
    public void afterLog(JoinPoint joinPoint) throws Exception {

        String controllerMethodDescription = getControllerMethodDescription(joinPoint);
        long beginTime = threadLocal.get().getTime();
        long endTime=System.currentTimeMillis();
        LOGGER.info("type:{},method:{},time:{}","after",controllerMethodDescription,endTime-beginTime);
    }


    @Around(value = "exec()")
    public Object aroundTest(ProceedingJoinPoint point){
        LOGGER.info("type:{}","around begin");
        Object o=null;
        try {
            o=point.proceed();
            LOGGER.info("type:{},name:{}","around",(String)o);
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        String modified=o+"modified";
        LOGGER.info("type:{},name:{}","around end",modified);
        return modified;
    }


    public String getControllerMethodDescription(JoinPoint joinPoint) throws Exception{
        //获取目标类名
        String targetName = joinPoint.getTarget().getClass().getName();
        //获取方法名
        String methodName = joinPoint.getSignature().getName();
        return targetName+"#"+methodName;
    }



    public static String getIpAddr(HttpServletRequest request) {
        String ip = request.getHeader("x-forwarded-for");
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
            if (ip.equals("127.0.0.1")) {
                //根据网卡取本机配置的IP
                InetAddress inet = null;
                try {
                    inet = InetAddress.getLocalHost();
                } catch (UnknownHostException e) {
                    e.printStackTrace();
                }
                ip = inet.getHostAddress();
            }
        }
        // 对于通过多个代理的情况，第一个IP为客户端真实IP,多个IP按照','分割
        if (ip != null && ip.length() > 15) {
            if (ip.indexOf(",") > 0) {
                ip = ip.substring(0, ip.indexOf(","));
            }
        }
        return ip;
    }
}

@RestController
public class MyController {

    @GetMapping("/name")
    public String print(){
        System.out.println("getName...");
        return "baozi";
    }
}
```

#### 通知的类型

| 通知            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| @Before         | 在一个方法执行之前，执行通知。                               |
| @After          | 在一个方法执行之后，不考虑其结果，执行通知。                 |
| @AfterReturning | 在切入点return内容之后切入内容（可以用来对处理返回值做一些加工处理） |
| @AfterThrowing  | 只有在方法退出抛出异常时，才能执行通知。                     |
| @Around         | 在切入点前后切入内容，并自己控制何时执行切入点自身的内容     |

### Spring AOP

可以通过配置文件或者编程的方式来使用Spring AOP。

配置可以通过xml文件来进行，大概有四种方式：

1.        配置ProxyFactoryBean，显式地设置advisors, advice, target等

2.        配置AutoProxyCreator，这种方式下，还是如以前一样使用定义的bean，但是从容器中获得的其实已经是代理对象

3.        通过<aop:config>来配置

4.        通过<aop: aspectj-autoproxy>来配置，使用AspectJ的注解来标识通知及切入点

https://blog.csdn.net/RobertoHuang/article/details/70148474 

也可以直接使用ProxyFactory来以编程的方式使用Spring AOP，通过ProxyFactory提供的方法可以设置target对象, advisor等相关配置，最终通过 getProxy()方法来获取代理对象

#### Spring AOP代理对象的生成

Spring提供了两种方式来生成代理对象: JDKProxy和Cglib，具体使用哪种方式生成由AopProxyFactory根据AdvisedSupport对象的配置来决定。默认的策略是如果目标类是接口，则使用JDK动态代理技术，否则使用Cglib来生成代理。下面我们来研究一下Spring如何使用JDK来生成代理对象，具体的生成代码放在JdkDynamicAopProxy这个类中，直接上相关代码：

```java
public Object getProxy(@Nullable ClassLoader classLoader) {
    if (logger.isTraceEnabled()) {
        logger.trace("Creating JDK dynamic proxy: " + this.advised.getTargetSource());
    }

    Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
    this.findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
    return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
}
```

我们知道InvocationHandler是JDK动态代理的核心，生成的代理对象的方法调用都会委托到InvocationHandler.invoke()方法。而通过JdkDynamicAopProxy的签名我们可以看到这个类其实也实现了InvocationHandler，下面我们就通过分析这个类中实现的invoke()方法来具体看下Spring AOP是如何织入切面的

```
@Nullable
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    Object oldProxy = null;
    boolean setProxyContext = false;
    TargetSource targetSource = this.advised.targetSource;
    Object target = null;

    Integer var9;
    try {
        if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
            Boolean var19 = this.equals(args[0]);
            return var19;
        }

        if (this.hashCodeDefined || !AopUtils.isHashCodeMethod(method)) {
            if (method.getDeclaringClass() == DecoratingProxy.class) {
                Class var18 = AopProxyUtils.ultimateTargetClass(this.advised);
                return var18;
            }

            Object retVal;
            if (!this.advised.opaque && method.getDeclaringClass().isInterface() && method.getDeclaringClass().isAssignableFrom(Advised.class)) {
                retVal = AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);
                return retVal;
            }

            if (this.advised.exposeProxy) {
                oldProxy = AopContext.setCurrentProxy(proxy);
                setProxyContext = true;
            }

            target = targetSource.getTarget();
            Class<?> targetClass = target != null ? target.getClass() : null;
            List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
            if (chain.isEmpty()) {
                Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
                retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
            } else {
                MethodInvocation invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
                retVal = invocation.proceed();
            }

            Class<?> returnType = method.getReturnType();
            if (retVal != null && retVal == target && returnType != Object.class && returnType.isInstance(proxy) && !RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
                retVal = proxy;
            } else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
                throw new AopInvocationException("Null return value from advice does not match primitive return type for: " + method);
            }

            Object var13 = retVal;
            return var13;
        }

        var9 = this.hashCode();
    } finally {
        if (target != null && !targetSource.isStatic()) {
            targetSource.releaseTarget(target);
        }

        if (setProxyContext) {
            AopContext.setCurrentProxy(oldProxy);
        }

    }

    return var9;
}
```

主流程可以简述为：获取可以应用到此方法上的通知链（Interceptor Chain）,如果有,则应用通知,并执行joinpoint; 如果没有,则直接反射执行joinpoint。而这里的关键是通知链是如何获取的以及它又是如何执行的，下面逐一分析下。

 首先，从上面的代码可以看到，通知链是通过Advised.getInterceptorsAndDynamicInterceptionAdvice()这个方法来获取的,我们来看下这个方法的实现:

```java
public List<Object> getInterceptorsAndDynamicInterceptionAdvice(Method method, @Nullable Class<?> targetClass) {
    AdvisedSupport.MethodCacheKey cacheKey = new AdvisedSupport.MethodCacheKey(method);
    List<Object> cached = (List)this.methodCache.get(cacheKey);
    if (cached == null) {
        cached = this.advisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice(this, method, targetClass);
        this.methodCache.put(cacheKey, cached);
    }

    return cached;
}
```

可以看到实际的获取工作其实是由AdvisorChainFactory. getInterceptorsAndDynamicInterceptionAdvice()这个方法来完成的，获取到的结果会被缓存。

```java
public List<Object> getInterceptorsAndDynamicInterceptionAdvice(Advised config, Method method, @Nullable Class<?> targetClass) {
    AdvisorAdapterRegistry registry = GlobalAdvisorAdapterRegistry.getInstance();
    Advisor[] advisors = config.getAdvisors();
    List<Object> interceptorList = new ArrayList(advisors.length);
    Class<?> actualClass = targetClass != null ? targetClass : method.getDeclaringClass();
    Boolean hasIntroductions = null;
    Advisor[] var9 = advisors;
    int var10 = advisors.length;

    for(int var11 = 0; var11 < var10; ++var11) {
        Advisor advisor = var9[var11];
        if (advisor instanceof PointcutAdvisor) {
            PointcutAdvisor pointcutAdvisor = (PointcutAdvisor)advisor;
            if (config.isPreFiltered() || pointcutAdvisor.getPointcut().getClassFilter().matches(actualClass)) {
                MethodMatcher mm = pointcutAdvisor.getPointcut().getMethodMatcher();
                boolean match;
                if (mm instanceof IntroductionAwareMethodMatcher) {
                    if (hasIntroductions == null) {
                        hasIntroductions = hasMatchingIntroductions(advisors, actualClass);
                    }

                    match = ((IntroductionAwareMethodMatcher)mm).matches(method, actualClass, hasIntroductions);
                } else {
                    match = mm.matches(method, actualClass);
                }

                if (match) {
                    MethodInterceptor[] interceptors = registry.getInterceptors(advisor);
                    if (mm.isRuntime()) {
                        MethodInterceptor[] var17 = interceptors;
                        int var18 = interceptors.length;

                        for(int var19 = 0; var19 < var18; ++var19) {
                            MethodInterceptor interceptor = var17[var19];
                            interceptorList.add(new InterceptorAndDynamicMethodMatcher(interceptor, mm));
                        }
                    } else {
                        interceptorList.addAll(Arrays.asList(interceptors));
                    }
                }
            }
        } else if (advisor instanceof IntroductionAdvisor) {
            IntroductionAdvisor ia = (IntroductionAdvisor)advisor;
            if (config.isPreFiltered() || ia.getClassFilter().matches(actualClass)) {
                Interceptor[] interceptors = registry.getInterceptors(advisor);
                interceptorList.addAll(Arrays.asList(interceptors));
            }
        } else {
            Interceptor[] interceptors = registry.getInterceptors(advisor);
            interceptorList.addAll(Arrays.asList(interceptors));
        }
    }

    return interceptorList;
}
```

这个方法执行完成后，Advised中配置能够应用到连接点或者目标类的Advisor全部被转化成了MethodInterceptor.

 

接下来我们再看下得到的拦截器链是怎么起作用的。

```
if (chain.isEmpty()) {
                Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
                retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
            } else {
                MethodInvocation invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
                retVal = invocation.proceed();
            }
```

​      

​         从这段代码可以看出，如果得到的拦截器链为空，则直接反射调用目标方法，否则创建MethodInvocation，调用其proceed方法，触发拦截器链的执行，来看下具体代码

```
@Nullable
    public Object proceed() throws Throwable {
        if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
            return this.invokeJoinpoint();
        } else {
            Object interceptorOrInterceptionAdvice = this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
            if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
                InterceptorAndDynamicMethodMatcher dm = (InterceptorAndDynamicMethodMatcher)interceptorOrInterceptionAdvice;
                Class<?> targetClass = this.targetClass != null ? this.targetClass : this.method.getDeclaringClass();
                return dm.methodMatcher.matches(this.method, targetClass, this.arguments) ? dm.interceptor.invoke(this) : this.proceed();
            } else {
                return ((MethodInterceptor)interceptorOrInterceptionAdvice).invoke(this);
            }
        }
    }
```

## 补充

### 静态代理

```java
public interface MyInterface {
    void print();
}

public class MyImplement implements MyInterface {
    @Override
    public void print() {
        System.out.println("hello world");
    }
}

public class MyProxy implements MyInterface{

    private MyImplement myImplement;

    public MyProxy() {
        this.myImplement = new MyImplement();
    }

    @Override
    public void print() {
        System.out.println("before");
        myImplement.print();
        System.out.println("after");
    }
}

public class Test {
    public static void main(String[] args) {
        MyInterface myInterface=new MyProxy();
        myInterface.print();
    }
}
```

这样的代理方式调用者不知道被代理对象的存在。

### 动态代理

从静态代理中可以看出，静态代理只能代理一个具体的类，如果要代理一个接口的多个实现的话需要定义不同的代理类，所以需要使用动态代理。

#### JDK实现

JDK动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。

`Proxy` 类是用于创建代理对象，而 `InvocationHandler` 接口来处理执行逻辑。 `Proxy` 的`newProxyInstance` 方法动态创建代理类。第一个参数为类加载器，第二个参数为代理类需要实现的接口列表，最后一个则是处理器。

```java
public class MyProxy implements InvocationHandler {

    private Object target;

    public MyProxy(Class clazz) {
        try {
            this.target=clazz.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before");
        Object result=method.invoke(target,args);
        System.out.println("after");
        return result;
    }

    public Object getProxy(){
        return Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(),
                target.getClass().getInterfaces(),this);
    }
}

public class Test {
    public static void main(String[] args) {
        MyProxy myProxy = new MyProxy(MyImplement.class);
        MyInterface myInterface= (MyInterface) myProxy.getProxy();
        myInterface.print();
    }
}
```

#### CGLIB实现

CGLIB是对一个小而快的字节码处理框架 `ASM` 的封装。把代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

```java
public class MyProxy implements MethodInterceptor {

    private Object target;

    public MyProxy(Class clazz) {
        try {
            this.target = clazz.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    public Object getProxy(){
        Enhancer enhancer=new Enhancer();
        enhancer.setSuperclass(target.getClass());
        enhancer.setCallback(this);
        return enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("before");
        Object result=method.invoke(target,objects);
        System.out.println("after");
        return result;
    }
}


public class Test {
    public static void main(String[] args) {
        MyInterface myInterface= (MyInterface) new MyProxy(MyImplement.class).getProxy();
        myInterface.print();
    }
}
```

**如何强制使用CGLIB实现AOP？**

1. 添加CGLIB库，SPRING_HOME/cglib/*.jar
2. 在spring配置文件中加入<aop:aspectj-autoproxy proxy-target-class=“true”/>

**JDK动态代理和CGLIB字节码生成的区别？**

* JDK动态代理只能对实现了接口的类生成代理，而不能针对类
* CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法，这就要求被该类或方法不要声明成final