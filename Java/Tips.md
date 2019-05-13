```java
Integer a1=64;
Integer a2=64;
Integer a3=128;
System.out.println((a1+a2)==a3);//true
```
+这个操作符不适用于Integer对象，首先两个Integer相加时会先进行自动拆箱操作，再进行数值相加

Integer对象无法与数值进行直接比较，所以会进行自动拆箱变成int

```java
String a="aaa";
String b="bbb";
String c="aaa"+"bbb";// 常量池对象
String d=a+b;// 堆内存对象
System.out.println(c==d);
```
**Java的泛型是如何工作的 ? 什么是类型擦除 ?**

泛型是通过类型擦除来实现的，编译器在编译时擦除了所有类型相关的信息，所以在运行时不存在任何类型相关的信息

**什么是泛型中的限定通配符和非限定通配符 ?**

　　这是另一个非常流行的Java泛型面试题。限定通配符对类型进行了限制。有两种限定通配符，一种是\<? extends T>它通过确保类型必须是T的子类来设定类型的上界，另一种是\<? super T>它通过确保类型必须是T的父类来设定类型的下界。泛型类型必须用限定内的类型来进行初始化，否则会导致编译错误。另一方面\<?>表示了非限定通配符，因为<?>可以用任意类型来替代。



方法重载是静态分派

方法重写是动态分派  (静态方法是静态分派，不能重写父类的静态方法)

域访问操作由编译器解析，不是多态的

静态代码块对于定义在它之后的静态变量，可以赋值，但是不能访问. 会报illegal forward reference





**Java是解析运行吗？**

不正确。Java源代码经过Javac编译成.class文件.class文件经JVM解析或编译运行。

- 解析:.class文件经过JVM内嵌的解析器解析执行。
- 编译:存在JIT编译器（即时编译器）把经常运行的代码作为"热点代码"编译与本地平台相关的机器码，并进行各种层次的优化。
- AOT(Ahead-of-Time Compilation)编译器: Java 9提供的直接将所有代码编译成机器码执行。

**final修饰成员变量**：

- final修饰类变量：必须要在**静态初始化块**中指定初始值或者**声明该类变量时**指定初始值，而且只能在这**两个地方**之一进行指定
- final修饰实例变量：必要要在**非静态初始化块**，**声明该实例变量**或者在**构造器中**指定初始值，而且只能在这**三个地方**进行指定
- 实例方法不能为final成员变量赋值

**Arrays.asList使用注意点**

- 在使用 asList 时不要将基本数据类型当做参数，asList接受的是泛型变长参数，8 个基本类型是无法作为 asList 的参数的， 要想作为泛型参数就必须使用其所对应的包装类型，但Java 中数组是一个对象，它是可以泛型化的

```java
int[] ints = {1,2,3,4,5};
List list = Arrays.asList(ints);
System.out.println(list.size());
Class<?> clazz = list.get(0).getClass();
System.out.println(clazz==ints.getClass());
```

- asList 产生的列表不可操作，aslist方法返回的是Arrays类中的一个内部类，并没有add等方法

```java
Integer[] ints = {1,2,3,4,5};
List<Integer> list = Arrays.asList(ints);
//list.add(6);
List<Integer> arrayList = new ArrayList<>(list);
arrayList.add(6);
arrayList.forEach(System.out::println);
```



1. 精度问题：2.0-1.1打印结果0.89999 因为二进制无法准确描述1/10   <9位用int  ； <18可以用long   ；>18必须使用bigdecimal。BigDecimal：add() 加； subtract()  减； multiply()  乘； divide()  除；构造是传String，否则还有精度问题

2. finally带return会覆盖返回值，否则finally里的赋值改变不了返回值

3. 接口中的变量都是public static final，方法都是public

4. public 所有类可见； protected 本包和子类可见； 默认 本包可见； private 本类可见

5. switch可以传 int short byte char string

6. 类初始化顺序：父类静态变量、静态语句块 → 子类静态变量、静态语句块 → 父类实例变量、普通语句块 → 父类构造方法 → 子类实例变量、普通语句块 → 子类构造方法

7. SpringCloud调试时经常出现服务非正常关闭导致端口被占用的情况，使用cmd使用`taskkill /f /t /im java.exe`杀进程

8. 如果父类没有实现序列化，而子类实现列序列化。那么父类中的成员没办法做序列化操作

**List Array互转**

```java
// array -> list
ArrayList<String> strList = new ArrayList<>(Arrays.asList("abc","def"));
// list -> array
String[] strings = strList.toArray(new String[strList.size()]);
```

**==与equals的区别**

==是判断两个对象的地址是不是相等  (基本数据类型比较的是值，引用数据类型比较的是内存地址) 

equals是判断两个变量或实例所指向的内存空间的值是不是相同，但它一般有两种使用情况：

- 情况1：类没有覆盖equals()方法。则通过equals()比较该类的两个对象时，等价于通过“==”比较这两个对象。
- 情况2：类覆盖了equals()方法。一般，我们都覆盖equals()方法来两个对象的内容相等；若它们的内容相等，则返回true(即，认为这两个对象相等)。

- String中的equals方法是被重写过的，因为object的equals方法是比较的对象的内存地址，而String的equals方法比较的是对象的值。





**hashCode equals**

- 如果两个对象相等，则hashcode一定也是相同的
- 两个对象相等,则equals方法都返回true
- 两个对象有相同的hashcode值，它们不一定是相等的,因为hashCode() 所使用的杂凑算法也许刚好会让多个对象传回相同的杂凑值
- equals方法被覆盖过，则hashCode方法也必须被覆盖
- hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写hashCode()，则该class的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

为什么重写equals时必须重写hashCode方法？

为什么要有hashCode

当你把对象加入HashSet时，HashSet会先计算对象的hashcode值来判断对象加入的位置，同时也会与其他已经加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals（）方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。。这样我们就大大减少了equals的次数，相应就大大提高了执行速度。



