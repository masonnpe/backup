---
title: 对比HashMap、Hashtable、LinkedHashMap、TreeMap
categories: Java
tags: [HashMap]
---

HashTable中的key、value都不能为null；HashMap中的key、value可以为null，很显然只能有一个key为null的键值对，但是允许有多个值为null的键值对；TreeMap中当未实现Comparator 接口时，key 不可以为null；当实现 Comparator 接口时，若未对null情况进行判断，则key不可以为null，反之亦然。
TreeMap是利用红黑树来实现的（树中的每个节点的值，都会大于或等于它的左子树种的所有节点的值，并且小于或等于它的右子树中的所有节点的值），实现了SortMap接口，能够对保存的记录根据键进行排序。所以一般需要排序的情况下是选择TreeMap来进行，默认为升序排序方式（深度优先搜索），可自定义实现Comparator接口实现排序方式。



HashMap基于哈希思想实现对数据的读写。当我们将键值对传递给put()方法时，它调用键对象的hashCode()方法来计算hashcode，让后找到bucket位置来储存值对象。当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。HashMap使用链表来解决碰撞问题，当发生碰撞了，对象将会储存在链表的下一个节点中。 HashMap在每个链表节点中储存键值对对象。当两个不同的键对象的hashcode相同时，它们会储存在同一个bucket位置的链表中，可通过键对象的equals()方法用来找到键值对。如果链表大小超过阈值（TREEIFY_THRESHOLD, 8），链表就会被改造为树形结构

扩容时：Hashtable将容量变为原来的2倍加1；HashMap扩容将容量变为原来的2倍`newCap = oldCap << 1`

HashMap不支持线程的同步，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致，在并发环境可能出现无限循环占用CPU、size不准确等诡异的问题。可以使用如下方法进行同步

1. 可以用 Collections的synchronizedMap方法
2. 使用ConcurrentHashMap类，相较于HashTable锁住的是对象整体， ConcurrentHashMap基于lock实现锁分段技术。首先将Map存放的数据分成一段一段的存储方式，然后给每一段数据分配一把锁，当一个线程占用锁访问其中一个段的数据时，其他段的数据也能被其他线程访问。ConcurrentHashMap不仅保证了多线程运行环境下的数据访问安全性，而且性能上有长足的提升。







LinkedHashMap通常提供的是遍历顺序符合插入顺序，它的实现是通过为条目（键值对）维护一个双向链表。注意，通过特定构造函数，我们可以创建反映访问顺序的实例，所
谓的put、get、compute等，都算作“访问”。
这种行为适用于一些特定应用场景，例如，我们构建一个空间占用敏感的资源池，希望可以自动将最不常被访问的对象释放掉，这就可以利用LinkedHashMap提供的机制来实现

```
import java.util.LinkedHashMap;
import java.util.Map;
public class LinkedHashMapSample {
 public satic void main(String[] args) {
 LinkedHashMap<String, String> accessOrderedMap = new LinkedHashMap<>(16, 0.75F, true){
 @Override
 protected boolean removeEldesEntry(Map.Entry<String, String> eldes) { // 实现自定义删除策略，否则行为就和普遍Map没有区别
 return size() > 3;
 }
 };
 accessOrderedMap.put("Project1", "Valhalla");
 accessOrderedMap.put("Project2", "Panama");
 accessOrderedMap.put("Project3", "Loom");
 accessOrderedMap.forEach( (k,v) -> {
 Sysem.out.println(k +":" + v);
 });
 // 模拟访问
 accessOrderedMap.get("Project2");
 accessOrderedMap.get("Project2");
 accessOrderedMap.get("Project3");
 Sysem.out.println("Iterate over should be not afected:");
 accessOrderedMap.forEach( (k,v) -> {
 Sysem.out.println(k +":" + v);
 });
 // 触发删除
 accessOrderedMap.put("Project4", "Mission Control");
 Sysem.out.println("Oldes entry should be removed:");
 accessOrderedMap.forEach( (k,v) -> {// 遍历顺序不变
 Sysem.out.println(k +":" + v);
 });
 }
}
```



依据resize源码，不考虑极端情况（容量理论最大极限由MAXIMUM_CAPACITY指定，数值为 1<<30，也就是2的30次方），我们可以归纳为：
门限值等于（负载因子）x（容量），如果构建HashMap的时候没有指定它们，那么就是依据相应的默认常量值。
门限通常是以倍数进行调整 （newThr = oldThr << 1），我前面提到，根据putVal中的逻辑，当元素个数超过门限大小时，则调整Map大小。
扩容后，需要将老的数组中的元素重新放置到新的数组，这是扩容的一个主要开销来源。
3.容量、负载因子和树化
前面我们快速梳理了一下HashMap从创建到放入键值对的相关逻辑，现在思考一下，为什么我们需要在乎容量和负载因子呢？
这是因为容量和负载系数决定了可用的桶的数量，空桶太多会浪费空间，如果使用的太满则会严重影响操作的性能。极端情况下，假设只有一个桶，那么它就退化成了链表，完全不
能提供所谓常数时间存的性能。
既然容量和负载因子这么重要，我们在实践中应该如何选择呢？
如果能够知道HashMap要存取的键值对数量，可以考虑预先设置合适的容量大小。具体数值我们可以根据扩容发生的条件来做简单预估，根据前面的代码分析，我们知道它需要符合
计算条件：
极客时间
负载因子 * 容量 > 元素数量
所以，预先设置的容量需要满足，大于“预估元素数量/负载因子”，同时它是2的幂数，结论已经非常清晰了。
而对于负载因子，我建议：
如果没有特别需求，不要轻易进行更改，因为JDK自身的默认负载因子是非常符合通用场景的需求的。
如果确实需要调整，建议不要设置超过0.75的数值，因为会显著增加冲突，降低HashMap的性能。
如果使用太小的负载因子，按照上面的公式，预设容量值也进行调整，否则可能会导致更加频繁的扩容，增加无谓的开销，本身访问性能也会受影响。
我们前面提到了树化改造，对应逻辑主要在putVal和treeifyBin中。
fnal void treeifyBin(Node[] tab, int hash) {
int n, index; Node e;
if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
resize();
else if ((e = tab[index = (n - 1) & hash]) != null) {
//树化改造逻辑
}
}
上面是精简过的treeifyBin示意，综合这两个方法，树化改造的逻辑就非常清晰了，可以理解为，当bin的数量大于TREEIFY_THRESHOLD时：
如果容量小于MIN_TREEIFY_CAPACITY，只会进行简单的扩容。
如果容量大于MIN_TREEIFY_CAPACITY ，则会进行树化改造。
那么，为什么HashMap要树化呢？
本质上这是个安全问题。因为在元素放置过程中，如果一个对象哈希冲突，都被放置到同一个桶里，则会形成一个链表，我们知道链表查询是线性的，会严重影响存取的性能。
而在现实世界，构造哈希冲突的数据并不是非常复杂的事情，恶意代码就可以利用这些数据大量与服务器端交互，导致服务器端CPU大量占用，这就构成了哈希碰撞拒绝服务攻击，
国内一线互联网公司就发生过类似攻击事件