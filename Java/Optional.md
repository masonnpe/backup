---
title: Optional
categories: Java
tags: [Optional]
---

## Optional

Java应用中最常见的bug就是空指针异常

`Optional`仅仅是一个容器，可以存放T类型的值或者`null`。它提供了一些有用的接口来避免显式的`null`检查，可以参考Java 8官方文档了解更多细节。

| method    | description                                                  |
| --------- | ------------------------------------------------------------ |
| isPresent | 有值返回true 否则返回false                                   |
| get()     | 有值时返回值  没有抛出异常                                   |
| orElse    | 有值时返回值  没有返回默认值                                 |
| orElseGet | 有值时返回值  没有返回一个supplier接口生成的值               |
| map       | 只存在就执行mapping函数调用,以将现有Optional实例的值转换成新的值 |

<!--more-->

## 创建Optional

```java
Apple apple = new Apple();
// 空的optional
Optional<Apple> optApple=Optional.empty(); 
// 值为nul抛出异常
Optional<Apple> optApple1=Optional.of(apple);   
//  值为null 返回空的optional
Optional<Apple> optApple2=Optional.ofNullable(apple); 
```

