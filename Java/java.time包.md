---
title: java.time包
categories: Java
tags: [Lambda]
---

java8新加的java.time包

```
LocalDate localDate=LocalDate.of(2018,10,29);
System.out.println(localDate.getYear());
System.out.println(localDate.getMonth());
System.out.println(localDate.getDayOfMonth());
System.out.println(localDate.getDayOfWeek());
System.out.println(localDate.isLeapYear());
```

```
LocalTime localTime= LocalTime.of(19,01,45);
System.out.println(localTime.getHour());
System.out.println(localTime.getMinute());
System.out.println(localTime.getSecond());
```

```
LocalDate localDate=LocalDate.parse("2018-07-21");
LocalTime localTime=LocalTime.parse("19:03:44");
```

<!--more-->