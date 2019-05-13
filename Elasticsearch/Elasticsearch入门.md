---
title: Elasticsearch入门
categories: Elasticsearch
tags: [Elasticsearch]
---

 Elasticsearch是一个开源的高扩展的分布式全文检索引擎，它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据。 
Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单

## elasticsearch与solr比较

### solr
优点
1、Solr有一个更大、更成熟的用户、开发和贡献者社区。
2、支持添加多种格式的索引，如：HTML、PDF、微软 Office 系列软件格式以及 JSON、XML、CSV 等纯文本格式。
3、Solr比较成熟、稳定。
4、不考虑建索引的同时进行搜索，速度更快。
缺点
建立索引时，搜索效率下降，实时索引搜索效率不高。

<!-- more -->

### Elasticsearch
优点
1、Elasticsearch是分布式的。不需要其他组件，分发是实时的，被叫做”Push replication”。
2、Elasticsearch 完全支持 Apache Lucene 的接近实时的搜索。
4、Elasticsearch 采用 Gateway 的概念，使得完备份更加简单。
5、各节点组成对等的网络结构，某些节点出现故障时会自动分配其他节点代替其进行工作。
缺点
1、还不够自动，不适合当前新的Index Warmup API (参考：http://zhaoyanblog.com/archives/764.html)

总结：
1、当单纯的对已有数据进行搜索时，Solr更快。
2、当实时建立索引时, Solr会产生io阻塞，查询性能较差, Elasticsearch具有明显的优势。
3、随着数据量的增加，Solr的搜索效率会变得更低，而Elasticsearch却没有明显的变化。
4、Solr的架构不适合实时搜索的应用。
5、Solr 支持更多格式的数据，而 Elasticsearch 仅支持json文件格式
6、Solr 在传统的搜索应用中表现好于 Elasticsearch，但在处理实时搜索应用时效率明显低于 Elasticsearch
7、Solr 是传统搜索应用的有力解决方案，但 Elasticsearch 更适用于新兴的实时搜索应用  

## 资料

- [《Elasticsearch学习，请先看这一篇！》](https://blog.csdn.net/laoyang360/article/details/52244917)
- [《Elasticsearch索引原理》](https://blog.csdn.net/cyony/article/details/65437708)

- [死磕es](https://blog.csdn.net/column/details/deep-elasticsearch.html)

- [es](https://blog.csdn.net/laoyang360/article/details/79293493)





### elasticsearch资源汇总

- [awesome-elasticsearch](https://github.com/dzharii/awesome-elasticsearch)
- [插件汇总](http://my.oschina.net/secisland/blog/636213)
- [ES权威指南中文](http://es.xiaoleilu.com/)
- [ES权威指南英文](https://www.elastic.co/guide/en/elasticsearch/guide/current/getting-started.html)
- [ES安全search-guard](https://github.com/floragunncom/search-guard)
- [elasticsearch相关介绍](http://www.searchtech.pro/)
- [solr和elasticsearch对比](https://thinkbiganalytics.com/solr-vs-elastic-search/)
- [将 ELASTICSEARCH 写入速度优化到极限](https://www.easyice.cn/archives/207)



### 剖析Elasticsearch集群系列

- [英文原文](http://insightdataengineering.com/blog/elasticsearch-crud/)
- [剖析Elasticsearch集群系列第一篇 存储模型和读写操作](http://www.infoq.com/cn/articles/analysis-of-elasticsearch-cluster-part01?utm_campaign=rightbar_v2&utm_source=infoq&utm_medium=articles_link&utm_content=link_text)
- [剖析Elasticsearch集群系列第二篇 分布式的三个C、translog和Lucene段](http://www.infoq.com/cn/articles/anatomy-of-an-elasticsearch-cluster-part02)
- [剖析Elasticsearch集群系列第三篇 近实时搜索、深层分页问题和搜索相关性权衡之道](http://www.infoq.com/cn/articles/anatomy-of-an-elasticsearch-cluster-part03?utm_campaign=rightbar_v2&utm_source=infoq&utm_medium=articles_link&utm_content=link_text) 
- [nested 和parent使用介绍](https://segmentfault.com/a/1190000002803966) 

### 企业使用案例

- [使用Akka、Kafka和ElasticSearch等构建分析引擎](http://www.infoq.com/cn/articles/use-akka-kafka--build-analysis-engine?utm_source=infoq&utm_medium=related_content_link&utm_campaign=relatedContent_articles_clk)
- [用Elasticsearch构建电商搜索平台，一个极有代表性的基础技术架构和算法实践案例](http://chuansong.me/n/690173551706)
- [用Elasticsearch+Redis构建投诉监控系统，看Airbnb如何保证用户持续增长](http://h2ex.com/1584) 
- [基于Elasticsearch构建千亿流量日志搜索平台实战](http://www.10tiao.com/html/175/201707/2653548856/1.html)
- [Elasticsearch作为时间序列数据库](http://blog.csdn.net/jiao_fuyou/article/details/49663687)

### 索引原理

- [ES原理](http://www.shaheng.me/blog/2015/06/elasticsearch--.html)
- [docvalues 介绍](http://qindongliang.iteye.com/blog/2297280)
- [ElasticSearch存储文件解析](https://www.elastic.co/blog/found-dive-into-elasticsearch-storage)

### 问题解决

- [如何防止elasticsearch的脑裂问题](https://segmentfault.com/a/1190000004504225)

### jar conflic

- [通过maven-shade-plugin 解决Elasticsearch与hbase的jar包冲突问题](http://blog.csdn.net/sunshine920103/article/details/51659936)