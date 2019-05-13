架构





并行度：Worker->Executor->Task，没错，是Task

流分组：Task与Task之间的数据流向关系

Shuffle Grouping：随机发射，负载均衡
Fields Grouping：根据某一个，或者某些个，fields，进行分组，那一个或者多个fields如果值完全相同的话，那么这些tuple，就会发送给下游bolt的其中固定的一个task

你发射的每条数据是一个tuple，每个tuple中有多个field作为字段

比如tuple，3个字段，name，age，salary

{"name": "tom", "age": 25, "salary": 10000} -> tuple -> 3个field，name，age，salary

All Grouping
Global Grouping
None Grouping
Direct Grouping
Local or Shuffle Grouping