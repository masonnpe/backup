---
title: LinkedHashMap实现一个LRU本地缓存
categories: Java
tags: [LinkedHashMap]
---

基于访问的最近最少使用算法

构造函数accessOrder=true时,当get（Object key）时，将最新访问的元素放到双向链表的第一位

<!--more-->

```
public class Demo {

	public static void main(String[] args) {
		Map<String,String> map=new LinkedHashMap<>(16,0.75f,true);
		map.put("brand1", "12");
		map.put("brand2", "13");
		map.put("brand3", "14");
		map.put("brand4", "15");
		map.put("brand5", "16");
		map.put("brand6", "17");
		print(map);
		map.get("brand2");
		print(map);
		map.get("brand5");
		print(map);
	}
	
	private static void print(Map<String,String> map) {
		Set<Entry<String,String>> set=map.entrySet();
		Iterator<Entry<String,String>> i=set.iterator();
		while(i.hasNext()) {
			Entry<String,String> entry=i.next();
			System.out.println(entry.getValue()+entry.getKey());
		}
		System.out.println("---");
	}
}
```

