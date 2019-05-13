---
title: Scala入门
categories: Scala
tags: [Scala]
---

Scala是一个面向对象和函数式编程的语言，运行于JVM之上，兼容Java程序

结尾不写；和Python有点像

类型声明，推测和Go有点像

scalac xxx.scala 编译Scala文件，生成Class字节码文件

scala xxx 运行字节码文件

卧槽和Java好像

<!--more-->

scala没有原生类型

访问修饰符与Java不同

- protected 只能被子类访问
- private  只能内部可见    外部类调用不了内部类的私有方法

## val 和 var 区别

var 变量

val 常量相当于final

## 基本数据类型

Byte/Char

Short/Int/Long/Float/Double

Boolean 

转换类型asInstanceOf[Double]

判断类型asInstanceOf[Double]

## Lazy

使用lazy可以延迟加载，第一次使用时，对应的表达式才会计算

lazy var a=10