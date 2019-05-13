---
title: Scala集合
categories: Scala
tags: [Scala]
---

## 数组  

new Array[]()   

Array()    

```
var arr=new Array[String](5)
arr(1)="hello"
println(arr.length)
var arr2=Array("hello","world")
println(arr2.mkString("-"))
var d=scala.collection.mutable.ArrayBuffer[String]()       //  可变的
d += "a"
d += ("1","2","3")
d ++= arr                                             // 加其他数组
d.insert(5,"mason")                                   // 插入指定位置
println(d)
remove
trimend
```

## list

Nil=空的list

```
var l=List(1,23,4,5,67)
println(l.head)        //第一个元素
println(l.tail)        //除了第一个
var l2=1::Nil          //1是头  Nil是尾

var lb=ListBuffer[Int]()  // 可变list
lb+=(1,23,5,4)
println(lb)
```

set

map

option

some

none

tuple