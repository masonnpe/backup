---
title: ApplicationContext
categories: Spring
tags: [ApplicationContext]
---

applicationcontext和beanfactory都是加载bean的

applicationcontext包含beanfactory的所有功能，是对beanfactory的扩展。   添加了@Qualifier  @Autowired等功能

beanfactory适合内存小的

<!--more-->

实例化bean比较复杂，FactoryBean是一个工厂类接口，可以通过改接口实例化bean的逻辑



spring中的循环依赖分三种

* 构造器循环依赖     无法解决 只能跑出beancurrentlyincreateionexception

* setter注入  通过提前暴露单例工厂方法addSingletionFactory

* protootype,spring无法完成依赖注入

