---
title: JVM性能监控和故障处理工具
categories: JVM
tags: [JVM]
---

- jps查看虚拟机进程状态  -l  主类的全名  -v  数据jvm参数
- jstat看虚拟机状态  jstat -gc 6424  jstat -gcutil 6424
- jinfo 看jvm的参数  jinfo -flag CMSInitiatingOccupancyFraction 5192
- jmap -dump:format=b,file=D:\DUMP.bin 5192  生成堆转储快照  jmap -heap 5192  堆详情信息
- jhat D:\DUMP.bin  分析dump
- jstack 5192 打印堆栈信息
- jconsole、visualVM可视化工具

<!--more-->