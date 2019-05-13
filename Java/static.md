---
title: static
categories: Java
tags: [static]
---

**静态变量**

- 静态变量属于类，可以直接通过类名来访问它
- 类的实例共享静态变量

**静态方法**

- 静态方法必须有实现
- 只能访问所属类的静态变量和静态方法，且方法中不能有 this 和 super 关键字

<!--more-->

**静态语句块**

- 静态语句块只会在类初始化时运行一次

**静态内部类**

- 非静态内部类依赖于外部类的实例，而静态内部类不需要
- 静态内部类不能访问外部类的非静态的变量和方法

**静态导包**

- 在使用静态变量和方法时不用再指明类名

```java
import static java.util.concurrent.Executors.*;

public abstract class Solution {
	public static void main(String[] args) {
		newCachedThreadPool();
	}
}
```

