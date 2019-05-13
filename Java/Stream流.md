---
title: Stream流
categories: Java
tags: [Stream]
---

## Stream

Java8的新增的Stream API（java.util.stream）将生成环境的函数式编程引入了Java库中。这是目前为止最大的一次对Java库的完善，以便开发者能够写出更加有效、更加简洁和紧凑的代码。Steam API极大得简化了集合操作，Stream流配合Lambda表达式使用起来真是美滋滋，steam的另一个价值是创造性地支持并行处理。

优点：性能高，灵活，表达清楚

## 方法

api分为两大类，中间操作相当于流水线，能对流水线上的东西进行各种处理，不会消耗流；终端操作消耗流，不能再对流进行操作

| 中间操作(流水线) |            |
| ---------------- | ---------- |
| filter           | 过滤       |
| map              | 抽取流     |
| limit            | 限制个数   |
| skip             | 跳过个数   |
| sorted           | 排序       |
| distinct         | 去重       |
| flagmap          | 合并到一起 |

| 终端操作(消耗流) |                  |
| ---------------- | ---------------- |
| collect          | 收集             |
| foreach          | 遍历             |
| count            | 计数             |
| anyMatch         | 至少匹配1个      |
| allMatch         | 匹配所有         |
| noneMatch        | 没有元素匹配     |
| findAny          | 返回流中任意元素 |
| reduce           |                  |

<!--more-->

## 举例

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class StreamTest {

    public static void main(String[] args) {
        List<Apple> appleList=new ArrayList<>();
        appleList.parallelStream().filter((e)->e.getWeight()>50)  // 筛选重量>50的苹果
                .sorted(Comparator.comparing(Apple::getColor))    // 根据苹果的颜色进行排序
                .map(Apple::getWeight)                            // 抽取苹果中的重量
                .limit(3)                                         // 限定选3个
                .distinct()                                       // 去掉相同的元素
                .collect(Collectors.toList());                    // 转化成list

        appleList.forEach(System.out::println); // 打印每个苹果
        appleList.stream().count();  // 计算苹果的总个数
        
		// 对list根据苹果的颜色进行分组
        Map<String,List<Apple>> map=appleList.parallelStream().collect(Collectors.groupingBy(Apple::getColor));

    }
}
```

flagmap合并流

```java
public class StreamTest {

    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("hello");
        names.add("world");
        List<String> a=names.stream().map((String e)->e.split(""))
                .flatMap(Arrays::stream)
                .collect(Collectors.toList());// 将两个单词，拆成单个字母并存为list
        System.out.println(a.get(9));
    }
}

```

reduce

```java
public class StreamTest {

    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("hello");
        names.add("world");
        String newstr=names.stream().map((String e)->e.split(""))
                .flatMap(Arrays::stream)
                .reduce("",String::concat);// 拆成字母后，拼接成一个新的字符串，相当于.reduce("",(String a ,String b)->a+b);
        System.out.println(newstr);
    }
}
```

collect

```java
List<Apple> appleList=new ArrayList<>();
appleList.stream().collect(Collectors.averagingInt(Apple::getWeight));// 计算重量的平均值
appleList.stream().map(Apple::getColor).collect(Collectors.joining());// 连接字符串
```

## 创建流的方式

**值创建流**

```
Stream<String>  strstream=Stream.of("dasd","dsa","das");
```

**数据创建流**

```
String[] nums={"qwe","ee","ds"};
Stream<String> aaa=Arrays.stream(nums);
```

**文件创建流**

```
try {
    Stream<String> lines=Files.lines(Paths.get("C:\\Users\\maruami\\Desktop\\readbook\\book.md"), Charset.defaultCharset());
    lines.forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}
```

**函数生成流**

```
Stream.generate(Math::random).limit(10).forEach(System.out::println);
```