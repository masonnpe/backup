---
title: Set
categories: Java
tags: [Set]
---

特点：不会存储重复的元素

## HashSet

Object的hashCode方法的话，hashCode会返回每个对象特有的序号（Java是依据对象的内存地址计算出的此序号），所以两个不同的对象的hashCode值是不可能相等的。

如果想要让两个不同的Object对象视为相等的，就必须重写类的hashCode方法和equals方法，同时也需要两个不同对象比较equals方法会返回true

HashSet和ArrayList都有判断元素是否相同的方法

- ArrayList使用boolean contains(Object o)-->indexOf(Object o)-->equals

- HashSet使用hashCode和equals方法

  - 通过hashCode方法和equals方法来保证元素的唯一性，add()返回的是boolean类型
  - 判断两个元素是否相同，先判断元素的hashCode值是否一致，如果相同才会去判断equals方法

<!--more-->

## TreeSet

基于红黑树实现，不能重复存储元素，还能实现排序

有两种自定义排序规则的方法

- 让存入的元素自身具有比较性
  - 实现Comparable接口，重写compareTo(Object o)方法
- 给TreeSet指定排序规则
  - 定义一个类实现接口Comparator，重写compare(Object o1, Object o2）方法，该接口的子类实例对象作为参数传递给TreeMap集合的构造方法
  - 如果Comparable和Comparator同时存在，优先Comparator

## LinkedHashSet

会保存与元素插入的顺序

## 总结

看到array，就要想到下标

看到link，就要想到first，last

看到hash，就要想到hashCode,equals

看到tree，就要想到两个接口。Comparable，Comparator

