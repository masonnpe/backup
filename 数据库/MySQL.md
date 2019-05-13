优化

- [《mysql数据库死锁的产生原因及解决办法》](https://www.cnblogs.com/sivkun/p/7518540.html)

### 索引

#### 聚集索引, 非聚集索引

- [《MyISAM和InnoDB的索引实现》](https://www.cnblogs.com/zlcxbb/p/5757245.html)

#### 复合索引

- [《复合索引的优点和注意事项》](https://www.cnblogs.com/summer0space/p/7247778.html)







[干货：mysql索引的数据结构](https://www.jianshu.com/p/1775b4ff123a)

[MySQL优化系列（三）--索引的使用、原理和设计优化](https://blog.csdn.net/Jack__Frost/article/details/72571540)

[数据库两大神器【索引和锁】](https://juejin.im/post/5b55b842f265da0f9e589e79#comment)

-  详细内容可以参考：   [可能是最漂亮的Spring事务管理详解](https://blog.csdn.net/qq_34337272/article/details/80394121)





### 数据存储

- [说说 SQL 优化之道](http://blog.720ui.com/2017/mysql_core_03_how_use_index/)
- MySQL 遇到的死锁问题
  - https://www.cnblogs.com/zejin2008/p/5262751.html
- [存储引擎的 InnoDB 与 MyISAM](http://blog.720ui.com/2017/mysql_core_02_innodb_myisam/)
- 数据库索引的原理
  - https://blog.csdn.net/suifeng3051/article/details/52669644
- limit 20000 加载很慢怎么解决
  - http://ourmysql.com/archives/1451





   limit 200000,10   相当于要200010行 舍弃 200000行（第一个参数为起始行，从0开始数；第二个参数为返回的总行数）

- 倒排索引

  - https://www.cnblogs.com/zlslch/p/6440114.html

--返回第3-5行  第一个参数为起始行，从0开始数；第二个参数为返回的总行数
SELECT * FROM mytable LIMIT 2, 3;