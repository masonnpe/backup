---
title: Lambda表达式
categories: Java
tags: [Lambda]
---

## Lambda表达式

Lambda 表达式，它是推动 Java 8 发布的最重要新特性。Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中），可以使代码变的更加简洁紧凑。

### 基本语法：

```
(参数列表) -> {代码块}
```

需要注意：

- 参数类型可省略，编译器可以自己推断
- 如果只有一个参数，圆括号可以省略
- 代码块如果只是一行代码，大括号也可以省略
- 如果代码块是一行，且是有结果的表达式，`return`可以省略

**注意：**事实上，把Lambda表达式可以看做是匿名内部类的一种简写方式。当然，前提是这个匿名内部类对应的必须是接口，而且接口中必须只有一个函数！Lambda表达式就是直接编写函数的：参数列表、代码体、返回值等信息。

<!--more-->

### 用法示例

#### 排序

```java
// 准备一个集合
List<Integer> list = Arrays.asList(10, 5, 25, -15, 20);
list.sort((i1,i2) -> { return i1 - i2;});
System.out.println(list);
//output [-15, 5, 10, 20, 25]
```

还可以简写为：

```java
list.sort((i1,i2) -> i1 - i2);
```

#### 遍历

```java
list.forEach(i -> System.out.println(i));
```

#### 赋值

Lambda表达式的实质其实还是匿名内部类，所以我们其实可以把Lambda表达式赋值给某个变量。

```java
Runnable task = () -> {System.out.println("hello lambda!");};
```

#### 隐式final

Lambda表达式的实质其实还是匿名内部类，而匿名内部类在访问外部局部变量时，要求变量必须声明为`final`。不过我们在使用Lambda表达式时无需声明`final`，因为Lambda底层会隐式的把变量设置为`final`，在后续的操作中，一定不能修改该变量。

```java
// 定义一个局部变量
int num = -1;
// 在Lambda表达式中使用局部变量num，num会被隐式声明为final，不能进行任何修改操作
Runnable r = () -> {System.out.println(num);};
```

## 函数式接口

- Lambda表达式是接口的匿名内部类的简写形式
- 接口必须满足：内部只有一个函数

其实这样的接口，我们称为函数式接口，我们学过的`Runnable`、`Comparator`都是函数式接口的典型代表。但是在实践中，函数接口是非常脆弱的，只要有人在接口里添加多一个方法，那么这个接口就不是函数接口了，就会导致编译失败。Java 8提供了一个特殊的注解`@FunctionalInterface`来克服上面提到的脆弱性并且显示地表明函数接口。而且jdk8版本中，对很多已经存在的接口都添加了`@FunctionalInterface`注解，例如`Runnable`接口，另外，Jdk8默认提供了一些函数式接口供我们使用：

### Function类型接口

```java
@FunctionalInterface
public interface Function<T, R> {
	// 接收一个参数T，返回一个结果R
    R apply(T t);
}
```

Function代表的是有参数，有返回值的函数。还有很多类似的Function接口：

| 接口名                 | 描述                                            |
| :--------------------- | ----------------------------------------------- |
| `BiFunction<T,U,R>`    | 接收两个T和U类型的参数，并且返回R类型结果的函数 |
| `DoubleFunction<R>`    | 接收double类型参数，并且返回R类型结果的函数     |
| `IntFunction<R>`       | 接收int类型参数，并且返回R类型结果的函数        |
| `LongFunction<R>`      | 接收long类型参数，并且返回R类型结果的函数       |
| `ToDoubleFunction<T>`  | 接收T类型参数，并且返回double类型结果           |
| `ToIntFunction<T>`     | 接收T类型参数，并且返回int类型结果              |
| `ToLongFunction<T>`    | 接收T类型参数，并且返回long类型结果             |
| `DoubleToIntFunction`  | 接收double类型参数，返回int类型结果             |
| `DoubleToLongFunction` | 接收double类型参数，返回long类型结果            |

这些都是一类函数接口，在Function基础上衍生出的，要么明确了参数不确定返回结果，要么明确结果不知道参数类型，要么两者都知道。

### Consumer

```java
@FunctionalInterface
public interface Consumer<T> {
	// 接收T类型参数，不返回结果
    void accept(T t);
}
```

具备类似的特征：那就是不返回任何结果。

### Predicate

```java
@FunctionalInterface
public interface Predicate<T> {
	// 接收T类型参数，返回boolean类型结果
    boolean test(T t);
}
```

Predicate系列参数不固定，但是返回的一定是boolean类型。

### Supplier

```java
@FunctionalInterface
public interface Supplier<T> {
	// 无需参数，返回一个T类型结果
    T get();
}
```

不接受任何参数，返回T类型结果。

## 方法引用

方法引用使得开发者可以将已经存在的方法作为变量来传递使用。方法引用可以和Lambda表达式配合使用。

### 语法

| 语法                   | 描述                             |
| ---------------------- | -------------------------------- |
| 类名::静态方法名       | 类的静态方法的引用               |
| 类名::非静态方法名     | 类的非静态方法的引用             |
| 实例对象::非静态方法名 | 类的指定实例对象的非静态方法引用 |
| 类名::new              | 类的构造方法引用                 |

### 示例

首先我们编写一个集合工具类，提供一个方法：

```java
    public class CollectionUtil{
        /**
         * 利用function将list集合中的每一个元素转换后形成新的集合返回
         * @param list 要转换的源集合
         * @param function 转换元素的方式
         * @param <T> 源集合的元素类型
         * @param <R> 转换后的元素类型
         * @return
         */
        public static <T,R> List<R> convert(List<T> list, Function<T,R> function){
            List<R> result = new ArrayList<>();
            list.forEach(t -> result.add(function.apply(t)));
            return result;
        }
    }
```

可以看到这个方法接收两个参数：

- `List<T> list`：需要进行转换的集合
- `Function<T,R>`：函数接口，接收T类型，返回R类型。用这个函数接口对list中的元素T进行转换，变为R类型

#### 类的静态方法引用

```java
List<Integer> list = Arrays.asList(1000, 2000, 3000);
```

我们需要把这个集合中的元素转为十六进制保存，需要调用`Integer.toHexString()`方法：

```java
public static String toHexString(int i) {
    return toUnsignedString0(i, 4);
}
```

这个方法接收一个 i 类型，返回一个`String`类型，可以用来构造一个`Function`的函数接口：

我们先按照Lambda原始写法，传入的Lambda表达式会被编译为`Function`接口，接口中通过`Integer.toHexString(i)`对原来集合的元素进行转换：

```java
// 通过Lambda表达式实现
List<String> hexList = CollectionUtil.convert(list, i -> Integer.toHexString(i));
System.out.println(hexList);// [3e8, 7d0, bb8]
```

上面的Lambda表达式代码块中，只有对`Integer.toHexString()`方法的引用，没有其它代码，因此我们可以直接把方法作为参数传递，由编译器帮我们处理，这就是静态方法引用：

```java
// 类的静态方法引用
List<String> hexList = CollectionUtil.convert(list, Integer::toHexString);
System.out.println(hexList);// [3e8, 7d0, bb8]
```

#### 类的非静态方法引用

接下来，我们把刚刚生成的`String`集合`hexList`中的元素都变成大写，需要借助于String类的toUpperCase()方法：

```java
public String toUpperCase() {
    return toUpperCase(Locale.getDefault());
}
```

这次是非静态方法，不能用类名调用，需要用实例对象，因此与刚刚的实现有一些差别，我们接收集合中的每一个字符串`s`。但与上面不同然后`s`不是`toUpperCase()`的参数，而是调用者：

```java
// 通过Lambda表达式，接收String数据，调用toUpperCase()
List<String> upperList = CollectionUtil.convert(hexList, s -> s.toUpperCase());
System.out.println(upperList);// [3E8, 7D0, BB8]
```

因为代码体只有对`toUpperCase()`的调用，所以可以把方法作为参数引用传递，依然可以简写：

```java
// 类的成员方法
List<String> upperList = CollectionUtil.convert(hexList, String::toUpperCase);
System.out.println(upperList);// [3E8, 7D0, BB8]
```

#### 指定实例的非静态方法引用

下面一个需求是这样的，我们先定义一个数字`Integer num = 2000`，然后用这个数字和集合中的每个数字进行比较，比较的结果放入一个新的集合。比较对象，我们可以用`Integer`的`compareTo`方法:

```java
public int compareTo(Integer anotherInteger) {
    return compare(this.value, anotherInteger.value);
}
```

先用Lambda实现，

```java
List<Integer> list = Arrays.asList(1000, 2000, 3000);

// 某个对象的成员方法
Integer num = 2000;
List<Integer> compareList = CollectionUtil.convert(list, i -> num.compareTo(i));
System.out.println(compareList);// [1, 0, -1]
```

与前面类似，这里Lambda的代码块中，依然只有对`num.compareTo(i)`的调用，所以可以简写。但是，需要注意的是，这次方法的调用者不是集合的元素，而是一个外部的局部变量`num`，因此不能使用 `Integer::compareTo`，因为这样是无法确定方法的调用者。要指定调用者，需要用 `对象::方法名`的方式：

```java
// 某个对象的成员方法
Integer num = 2000;
List<Integer> compareList = CollectionUtil.convert(list, num::compareTo);
System.out.println(compareList);// [1, 0, -1]
```

#### 构造函数引用

最后一个场景：把集合中的数字作为毫秒值，构建出`Date`对象并放入集合，这里我们就需要用到Date的构造函数：

```java
/**
  * @param   date   the milliseconds since January 1, 1970, 00:00:00 GMT.
  * @see     java.lang.System#currentTimeMillis()
  */
public Date(long date) {
    fastTime = date;
}
```

我们可以接收集合中的每个元素，然后把元素作为`Date`的构造函数参数：

```java
// 将数值类型集合，转为Date类型
List<Date> dateList = CollectionUtil.convert(list, i -> new Date(i));
// 这里遍历元素后需要打印，因此直接把println作为方法引用传递了
dateList.forEach(System.out::println);
```

上面的Lambda表达式实现方式，代码体只有`new Date()`一行代码，因此也可以采用方法引用进行简写。但问题是，构造函数没有名称，我们只能用`new`关键字来代替：

```java
// 构造方法
List<Date> dateList = CollectionUtil.convert(list, Date::new);
dateList.forEach(System.out::println);
```

注意两点：

- 上面代码中的System.out::println 其实是 指定对象System.out的非静态方法println的引用
- 如果构造函数有多个，可能无法区分导致传递失败

