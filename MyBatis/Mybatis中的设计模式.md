---
title: Mybatis中的设计模式
categories: Mybatis
tags: []
---

一、装饰模式

最明显的就是cache包下面的实现

Cahe、LoggingCache、LruCache、TransactionalCahe...等

以LoggingCache为例，UML图

​                              



Cache cache  = new LoggingCache(new PerpetualCache("cacheid"));
一层层包装就使得默认cache实现PerpetualCache具有附加的功能，比如上面的log功能。
二、建造者模式

BaseBuilder、XMLMapperBuilder





三、工厂方法

SqlSessionFactory



四、适配器模式

Log、LogFactory



五、模板方法

BaseExecutor、SimpleExecutor



六、动态代理

Plugin 见7图

7、责任链模式

Interceptor、InterceptorChain
