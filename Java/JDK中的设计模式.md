---
title: JDK中的设计模式
categories: Java
tags: []
---

- 结构型模式：
  - 适配器：用来把一个接口转化成另一个接口，如 java.util.Arrays#asList()。java.io.InputStreamReader(InputStream)java.io.OutputStreamWriter(OutputStream)

  - 桥接模式：这个模式将抽象和抽象操作的实现进行了解耦，这样使得抽象和实现可以独立地变化，如JDBC；

  - 组合模式：使得客户端看来单个对象和对象的组合是同等的。换句话说，某个类型的方法同时也接受自身类型作为参数，如 Map.putAll，List.addAll、Set.addAll。

  - 装饰者模式：动态的给一个对象附加额外的功能，这也是子类的一种替代方式，如 java.util.Collections#checkedList|Map|Set|SortedSet|SortedMap。java.io.BufferedInputStream(InputStream)java.io.DataInputStream(InputStream)java.io.BufferedOutputStream(OutputStream)

  - 享元模式：使用缓存来加速大量小对象的访问时间，如 valueOf(int)。

  - 代理模式：代理模式是用一个简单的对象来代替一个复杂的或者创建耗时的对象，如 java.lang.reflect.Proxy


##### 门面模式：

    给一组组件，接口，抽象，或者子系统提供一个简单的接口。

  * java.lang.Class

  - javax.faces.webapp.FacesServlet

- 创建模式:

  - 抽象工厂模式：抽象工厂模式提供了一个协议来生成一系列的相关或者独立的对象，而不用指定具体对象的类型，如 java.util.Calendar#getInstance()。java.util.Arrays#asList()java.util.ResourceBundle#getBundle()java.sql.DriverManager#getConnection()java.sql.Connection#createStatement()java.sql.Statement#executeQuery()

  - 建造模式(Builder)：定义了一个新的类来构建另一个类的实例，以简化复杂对象的创建，如：java.lang.StringBuilder#append()。

  - 工厂方法：就是 **一个返*** 回具体对象的方法，而不是多个，如 java.lang.Object#toString()、java.lang.Class#newInstance()。java.lang.Class#newInstance()java.lang.Class#forName()

  - 原型模式：使得类的实例能够生成自身的拷贝、如：java.lang.Object#clone()。java.lang.Object#clone()java.lang.Cloneable

##### 单例模式

    用来确保类只有一个实例。
    
    java.lang.Runtime#getRuntime()

- 行为模式：

  - 责任链模式：通过把请求从一个对象传递到链条中下一个对象的方式，直到请求被处理完毕，以实现对象间的解耦。如 javax.servlet.Filter#doFilter()。 java.util.logging.Logger#log()

  - 命令模式：将操作封装到对象内，以便存储，传递和返回，如：java.lang.Runnable。

  - 解释器模式：定义了一个语言的语法，然后解析相应语法的语句，如，java.text.Format，java.text.Normalizer。

  - 迭代器模式：提供一个一致的方法来顺序访问集合中的对象，如 java.util.Iterator。

  - 中介者模式：通过使用一个中间对象来进行消息分发以及减少类之间的直接依赖，java.lang.reflect.Method#invoke()。java.util.Timerjava.util.concurrent.Executor#execute()java.util.concurrent.ExecutorService#submit()java.lang.reflect.Method#invoke()

  - 空对象模式：如 java.util.Collections#emptyList()。

  - 观察者模式：它使得一个对象可以灵活的将消息发送给感兴趣的对象，如 java.util.EventListener。javax.servlet.http.HttpSessionBindingListenerjavax.servlet.http.HttpSessionAttributeListenerjavax.faces.event.PhaseListener

  - 模板方法模式：让子类可以重写方法的一部分，而不是整个重写，如 java.util.Collections#sort()。



##### 备忘录模式

    生成对象状态的一个快照，以便对象可以恢复原始状态而不用暴露自身的内容。Date对象通过自身内部的一个long值来实现备忘录模式。
    
    java.util.Date
    
    java.io.Serializable

##### 状态模式

通过改变对象内部的状态，使得你可以在运行时动态改变一个对象的行为。

- - - java.util.Iterator
    - javax.faces.lifecycle.LifeCycle#execute()

##### 策略模式

使用这个模式来将一组算法封装成一系列对象。通过传递这些对象可以灵活的改变程序的功能。

- java.util.Comparator#compare()
- javax.servlet.http.HttpServlet
- javax.servlet.Filter#doFilter()

##### 模板方法模式

让子类可以重写方法的一部分，而不是整个重写，你可以控制子类需要重写那些操作。

* java.util.Collections#sort()

- java.io.InputStream#skip()
- java.io.InputStream#read()
- java.util.AbstractList#indexOf()

##### 访问者模式

提供一个方便的可维护的方式来操作一组对象。它使得你在不改变操作的对象前提下，可以修改或者扩展对象的行为。

- javax.lang.model.element.Element and javax.lang.model.element.ElementVisitor
- javax.lang.model.type.TypeMirror and javax.lang.model.type.TypeVisitor

