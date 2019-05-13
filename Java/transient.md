---
title: transient
categories: Java
tags: [transient]

---

对于实现`Serializable`接口的类，属性前添加关键字`transient`，序列化对象的时候，这个属性就不会被序列化

```java
transient Object[] elementData; 
```

## 注意点

- 要使`transient`生效，类需要实现`Serializable`接口
- `transient`关键字只能修饰变量，不能修饰方法和类
- 静态变量不管是否被`transient`修饰，均不能被序列化，反序列化后对象中`static`型变量的值为当前JVM中对应类中`static`变量的值

<!--more-->

**测试代码**

```java
    static class TransientDemo implements Serializable
    {
        private static final long serialVersionUID = -5818461504980738394L;
        private String name;
        private String password;
        private static String staticPassword;
        private transient static String transientAndStaticPassword;
		// get/set省略
    }

    public static void main(String[] args) {
        TransientDemo transientDemo = new TransientDemo();
        transientDemo.setName("root");
        transientDemo.setPassword("*********");
        transientDemo.setStaticPassword("*********");
        transientDemo.setTransientAndStaticPassword("*********");

        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        byte[] b = null;
        try {
            ObjectOutputStream oos = new ObjectOutputStream(baos);
            oos.writeObject(transientDemo);
            b = baos.toByteArray();
            System.err.println("before name=" + transientDemo.getName());
            System.err.println("before getPassword=" + transientDemo.getPassword());
            System.err.println("before getStaticPassword=" + transientDemo.getStaticPassword());
            System.err.println("before getTransientAndStaticPassword=" + transientDemo.getTransientAndStaticPassword());
            TransientDemo.staticPassword = "我不是真的";
            TransientDemo.transientAndStaticPassword = "我不是真的";
        }
        catch (IOException e) {
            e.printStackTrace();
        }
        ByteArrayInputStream bais = new ByteArrayInputStream(b);
        try {
            ObjectInputStream ois = new ObjectInputStream(bais);
            TransientDemo demo = (TransientDemo) ois.readObject();
            System.err.println("after name=" + demo.getName());
            System.err.println("after getPassword=" + demo.getPassword());
            System.err.println("after getStaticPassword=" + demo.getStaticPassword());
            System.err.println("after getTransientAndStaticPassword=" + demo.getTransientAndStaticPassword());
        }
        catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }

    }
```

**打印结果**

```java
before name=root
before getPassword=*********
before getStaticPassword=*********
before getTransientAndStaticPassword=*********
after name=root
after getPassword=*********
after getStaticPassword=我不是真的
after getTransientAndStaticPassword=我不是真的
```

## 自定义序列化方式

被`transient`修饰的属性可以通过`writeObject()`方法，自定义属性的序列化方式

参考JDK1.8 `ArrayList`自定义了对`elementData`的序列化与反序列化 

```java
transient Object[] elementData; // non-private to simplify nested class access

private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);    //只序列化被使用的数据
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }


private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();		 //反序列化
            }
        }
    }
```

## Externalizable

`Externalizable`和`Serializable`相比，`Externalizable`是`Serializable`接口的子类

- `Serializable`序列化时不会调用默认的构造器，而`Externalizable`序列化时会调用默认构造器的，否则会报`java.io.InvalidClassException`

- 实现了`Serializable`接口的类，默认会序列化对象的所有属性（包括`private`属性和引用的对象）

  `Externalizable`默认不序列化对象的任何属性，通过重写`writeExternal()`和`readExternal()`方法指定序列化哪些属性 