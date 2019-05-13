---
title: Scala函数
categories: Scala
tags: [Scala]
---

## 变长参数

```scala
object Demo {
  def main(args: Array[String]): Unit = {
    val s=sum(1,2,3,4,5)
    println(s)
  }

  def sum(nums: Int*): Int ={
    var result=0;
    for (num<- nums){
      result+=num
    }
    result
  }
}
```

也可以使用`val s=sum(1 to 5: _*)`使用序列调用变长参数

## 异常

```scala
object Demo {
  def main(args: Array[String]): Unit = {
    try {
      throw new IllegalArgumentException("exception")
    }catch {
      case _:IllegalArgumentException=>println("catch")
    }finally {
      println("finally")
    }
  }
}
```

<!--more-->

to	[x,y]

range   [x,y）  还有step步长

until  

_占位符

```
//指定范围
private [this] val gender="male"
```

```
class Animal(val name:String,val age:Int){
  var sch="jit";
  println(name+"age:"+age+"school"+sch)
  // 附属构造器
  def this( name:String, age:Int,sch:String){
    this(name,age)
    this.sch=sch
  }
}

```

## 继承+重写

```
class Tiger(name:String,age:Int,height:Int) extends Animal(name, age ){
  override def toString: String = {
    "重写了"
  }
}
class Animal( name:String, age:Int){
  var sch="jit";
  println(name+"age:"+age+"school"+sch)
  // 附属构造器
  def this( name:String, age:Int,sch:String){
    this(name,age)
    this.sch=sch
  }
}
```

## apply方法

单例对象与类同名时，这个单例对象被称为这个类的伴生对象，而这个类被称为这个单例对象的伴生类。伴生类和伴生对象要在同一个源文件中定义，伴生对象和伴生类可以互相访问其私有成员。不与伴生类同名的单例对象称为孤立对象

## String打印

```
object ArrayApp {
  def main(args: Array[String]): Unit = {
    val s="hello:pk"
    var name="mason"
    println(s"hello:$name")

    var b=
      """
        |
      """.stripMargin
    println(b)

  }
}
```

## 匿名函数和科里化

```
object ArrayApp {
  def main(args: Array[String]): Unit = {
    var a=(x:Int)=> x+1
    println(a(10))

    // curry 科里化
    println(sum(2)(1))

  }

  def sum(a:Int)(b:Int)=a+b
}
```

map filter flatmap reduce 