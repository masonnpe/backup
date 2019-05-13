---
title: Array和ArrayBuffer
categories: Scala
tags: [Scala]
---

Array固定长度  ArrayBuff可变

```scala
object Demo {
  def main(args: Array[String]): Unit = {
    // 初始化数组
    val a=new Array[Int](100)
    var b=Array("hello","world")

    // 初始化ArrayBuff可变数组 相当于arraylist
    var c=ArrayBuffer[Int]()
    // 添加元素
    c+=1
    // 添加多个元素
    c+=(2,3,4,5)
    // 添加其他集合的元素
    c++=Array(6,7,8,9)
    // 尾部去2个
    c.trimEnd(2)
    // 在某个索引位置 插入元素
    c.insert(0,10,11)
    // 删除某个位置的元素
    c.remove(0)
    // 索引位置  个数
    c.remove(0,3)
    // 遍历打印
    c.foreach(println)
    // array arraybuffer 可以互相转换
    c.toArray
    b.toBuffer
	// 求和
    a.sum
	// 最大值
    a.max
	// 排序
    Sorting.quickSort(a)
    // 获取数据所有元素内容
    val str=a.mkString("<",",",">");
    println(str)
  }
}
```

<!--more-->