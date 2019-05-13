倒排索引简单来说就是 把一些数据分词   以词为单位  保存数据的id

## 概念

index相当于数据库表

type 一个index里有多个type

mapping 表结构

document 表中记录

## es的分布式架构原理

一个索引可以拆成多个shard放部分数据  将备份shard放其他机器  数据写入primary同步到replication，要是master node宕机了，那么会重新选举一个节点为master node

（2）es写入数据的工作原理是什么啊？es查询数据的工作原理是什么啊？



 写

1. 客户端选择一个node发送请求过去，这个node就是coordinating node（协调节点）
2. coordinating node，对document进行路由，将请求转发给对应的node（有primary shard）
3. 实际的node上的primary shard处理请求，然后将数据同步到replica node
4. coordinating node，如果发现primary node和所有replica node都搞定后，就返回响应结果给客户端

读

​	查询，GET某一条数据，写入了某个document，这个document会自动给你分配一个全局唯一的id，doc id，同时也是根据doc id进行hash路由到对应的primary shard上面去。也可以手动指定doc id，比如用订单id，用户id。

 

你可以通过doc id来查询，会根据doc id进行hash，判断出来当时把doc id分配到了哪个shard上面去，从那个shard去查询

 

1）客户端发送请求到任意一个node，成为coordinate node

2）coordinate node对document进行路由，将请求转发到对应的node，此时会使用round-robin随机轮询算法，在primary shard以及其所有replica中随机选择一个，让读请求负载均衡

3）接收请求的node返回document给coordinate node

4）coordinate node返回document给客户端

 

es搜索数据过程

1. 客户端发送请求到一个coordinate node
2. 协调节点将搜索请求转发到所有的shard对应的primary shard或replica shard也可以
3. query phase：每个shard将自己的搜索结果（其实就是一些doc id），返回给协调节点，由协调节点进行数据的合并、排序、分页等操作，产出最终结果
4. fetch phase：接着由协调节点，根据doc id去各个节点上拉取实际的document数据，最终返回给客户端

 

（5）写数据底层原理

 

1. 数据先写入buffer，在buffer里的时候数据是搜索不到的；同时将数据写入translog日志文件
2. 如果buffer快满了，或者每隔一段时间（默认1秒），就会将buffer数据refresh到一个新的segment file中，但是此时数据不是直接进入segment file的磁盘文件的，而是先进入os cache的。这个过程就是refresh。数据写入磁盘文件之前，会先进入os cache，先进入操作系统级别的一个内存缓存中去。只要buffer中的数据被refresh操作，刷入os cache中，就代表这个数据就可以被搜索到了，所以是准实时的
3. 随着这个过程推进，translog会变得越来越大。当translog达到一定长度的时候，就会触发commit操作。

 

commit操作：

1、写commit point；将一个commit point写入磁盘文件，里面标识着这个commit point对应的所有segment file

2、将os cache数据fsync强刷到磁盘上去；

3、清空translog日志文件

 默认每隔30分钟会自动执行一次commit，但是如果translog过大，也会触发commit。整个commit的过程，叫做flush操作。我们可以手动执行flush操作，就是将所有os cache数据刷到磁盘文件中去



translog日志文件的作用是什么？就是在你执行commit操作之前，数据要么是停留在buffer中，要么是停留在os cache中，无论是buffer还是os cache都是内存，一旦这台机器死了，内存中的数据就全丢了。

 

所以需要将数据对应的操作写入一个专门的日志文件，translog日志文件中，一旦此时机器宕机，再次重启的时候，es会自动读取translog日志文件中的数据，恢复到内存buffer和os cache中去。

 

translog其实也是先写入os cache的，默认每隔5秒刷一次到磁盘中去，所以默认情况下，可能有5秒的数据会仅仅停留在buffer或者translog文件的os cache中，如果此时机器挂了，会丢失5秒钟的数据。但是这样性能比较好，最多丢5秒的数据。也可以将translog设置成每次写操作必须是直接fsync到磁盘，但是性能会差很多。

 



es丢数据的问题，   es第一是准实时的，数据写入1秒后可以搜索到；可能会丢失数据的，你的数据有5秒的数据，停留在buffer、translog os cache、segment file os cache中，有5秒的数据不在磁盘上，此时如果宕机，会导致5秒的数据丢失。

 

如果你希望一定不能丢失数据的话，你可以设置个参数，官方文档，百度一下。每次写入一条数据，都是写入buffer，同时写入磁盘上的translog，但是这会导致写性能、写入吞吐量会下降一个数量级。本来一秒钟可以写2000条，现在你一秒钟只能写200条，都有可能。

 

10）如果是删除操作，commit的时候会生成一个.del文件，里面将某个doc标识为deleted状态，那么搜索的时候根据.del文件就知道这个doc被删除了

 

11）如果是更新操作，就是将原来的doc标识为deleted状态，然后新写入一条数据

 

12）buffer每次refresh一次，就会产生一个segment file，所以默认情况下是1秒钟一个segment file，segment file会越来越多，此时会定期执行merge

 

13）每次merge的时候，会将多个segment file合并成一个，同时这里会将标识为deleted的doc给物理删除掉，然后将新的segment file写入磁盘，这里会写一个commit point，标识所有新的segment file，然后打开segment file供搜索使用，同时删除旧的segment file。

 

es里的写流程，有4个底层的核心概念，refresh、flush、translog、 merge

 

当segment file多到一定程度的时候，es就会自动触发merge操作，将多个segment file给merge成一个segment file。







































## es如何提高查询性能

 

![image](http://ws3.sinaimg.cn/large/007iUdjSgy1fy6pkfcvtgj30rd0ect8w.jpg)

 es其实性能并没有你想象中那么好的，数据量很大的情况下（数十亿级别）,跑个搜索怎么一下5秒~10秒，坑爹了。第一次搜索的时候，是5-10秒，后面反而就快了，可能就几百毫秒。

 

（1）性能优化的杀手锏——filesystem cache

 

os cache，操作系统的缓存

 

你往es里写的数据，实际上都写到磁盘文件里去了，磁盘文件里的数据操作系统会自动将里面的数据缓存到os cache里面去，es的搜索引擎严重依赖于底层的filesystem cache，你如果给filesystem cache更多的内存，尽量让内存可以容纳所有的indx segment file索引数据文件，那么你搜索的时候就基本都是走内存的，性能会非常高。生产环境实践经验，在es中就存少量的数据，就是你要用来搜索的那些索引，内存留给filesystem cache的，就100G，那么你就控制在100gb以内，相当于是，你的数据几乎全部走内存来搜索，性能非常之高，一般可以在1秒以内

 

比如说你现在有一行数据id name age ....30个字段但是你现在搜索，只需要根据id name age三个字段来搜索

 如果你傻乎乎的往es里写入一行数据所有的字段，就会导致说70%的数据是不用来搜索的，结果硬是占据了es机器上的filesystem cache的空间，单挑数据的数据量越大，就会导致filesystem cahce能缓存的数据就越少

仅仅只是写入es中要用来检索的少数几个字段就可以了，比如说，就写入es id name age三个字段就可以了，然后你可以把其他的字段数据存在mysql里面，我们一般是建议用es + hbase的这么一个架构。

 

hbase的特点是适用于海量数据的在线存储，就是对hbase可以写入海量数据，不要做复杂的搜索，就是做很简单的一些根据id或者范围进行查询的这么一个操作就可以了

 

从es中根据name和age去搜索，拿到的结果可能就20个doc id，然后根据doc id到hbase里去查询每个doc id对应的完整的数据，给查出来，再返回给前端。

 

你最好是写入es的数据小于等于，或者是略微大于es的filesystem cache的内存容量

elastcisearch减少数据量仅仅放要用于搜索的几个关键字段即可，尽量写入es的数据量跟es机器的filesystem cache是差不多的就可以了；其他不用来检索的数据放hbase里，或者mysql。从es检索根据es返回的id去hbase里查询

（2）数据预热

 

假如说，哪怕是你就按照上述的方案去做了，es集群中每个机器写入的数据量还是超过了filesystem cache一倍，比如说你写入一台机器60g数据，结果filesystem cache就30g，还是有30g数据留在了磁盘上。

每隔一会儿，你自己的后台系统去搜索一下热数据，刷到filesystem cache里去，后面用户实际上来看这个热数据的时候，他们就是直接从内存里搜索了，很快。

 

电商，你可以将平时查看最多的一些商品，比如说iphone 8，热数据提前后台搞个程序，每隔1分钟自己主动访问一次，刷到filesystem cache里去。

 

热点数据，最好做一个专门的缓存预热子系统，就是对热数据，每隔一段时间，你就提前访问一下，让数据进入filesystem cache里面去。这样期待下次别人访问的时候，一定性能会好一些。

 

（3）冷热分离

关于es性能优化，数据拆分，我之前说将大量不搜索的字段，拆分到别的存储中去，这个就是类似于后面我最后要讲的mysql分库分表的垂直拆分。

 

es可以做类似于mysql的水平拆分，就是说将大量的访问很少，频率很低的数据，单独写一个索引，然后将访问很频繁的热数据单独写一个索引

 

你最好是将冷数据写入一个索引中，然后热数据写入另外一个索引中，这样可以确保热数据在被预热之后，尽量都让他们留在filesystem os cache里，别让冷数据给冲刷掉。

 

你看，假设你有6台机器，2个索引，一个放冷数据，一个放热数据，每个索引3个shard

 

3台机器放热数据index；另外3台机器放冷数据index

 

然后这样的话，你大量的时候是在访问热数据index，热数据可能就占总数据量的10%，此时数据量很少，几乎全都保留在filesystem cache里面了，就可以确保热数据的访问性能是很高的。

 

但是对于冷数据而言，是在别的index里的，跟热数据index都不再相同的机器上，大家互相之间都没什么联系了。如果有人访问冷数据，可能大量数据是在磁盘上的，此时性能差点，就10%的人去访问冷数据；90%的人在访问热数据。

 

（4）document模型设计

 es里面的复杂的关联查询，复杂的查询语法，尽量别用，一旦用了性能一般都不太好，比如join，nested，parent-child

比如mysql有两张表

订单表：id order_num total_price

订单条目表：id order_id goods_id purchase_count price

我在mysql里，都是select * from order join order_item on order.id=order_item.order_id where order.id=1



写入es的时候，搞成两个索引，order索引，orderItem索引

 order索引，里面就包含id order_code total_price

orderItem索引，里面写入进去的时候，就完成join操作，id order_num total_price id order_id goods_id purchase_count price将关联好的数据直接写入es中，搜索的时候，就不需要利用es的搜索语法去完成join来搜索了

 

两个思路，在搜索/查询的时候，要执行一些业务强相关的特别复杂的操作：

 

1）在写入数据的时候，就设计好模型，加几个字段，把处理好的数据写入加的字段里面

2）自己用java程序封装，es能做的，用es来做，搜索出来的数据，在java程序里面去做，比如说我们，基于es，用java封装一些特别复杂的操作

 

（5）分页性能优化

 

es的分页是较坑的，为啥呢？举个例子吧，假如你每页是10条数据，你现在要查询第100页，实际上是会把每个shard上存储的前1000条数据都查到一个协调节点上，如果你有个5个shard，那么就有5000条数据，接着协调节点对这5000条数据进行一些合并、处理，再获取到最终第100页的10条数据。

 

分布式的，你要查第100页的10条数据，你是不可能说从5个shard，每个shard就查2条数据？最后到协调节点合并成10条数据？你必须得从每个shard都查1000条数据过来，然后根据你的需求进行排序、筛选等等操作，最后再次分页，拿到里面第100页的数据。

 

你翻页的时候，翻的越深，每个shard返回的数据就越多，而且协调节点处理的时间越长。非常坑爹。所以用es做分页的时候，你会发现越翻到后面，就越是慢。

 

我们之前也是遇到过这个问题，用es作分页，前几页就几十毫秒，翻到10页之后，几十页的时候，基本上就要5~10秒才能查出来一页数据了

 

1）不允许深度分页/默认深度分页性能很惨

 

你系统不允许他翻那么深的页，pm，默认翻的越深，性能就越差

 

2）类似于app里的推荐商品不断下拉出来一页一页的

 

类似于微博中，下拉刷微博，刷出来一页一页的，你可以用scroll api，自己百度

 

scroll会一次性给你生成所有数据的一个快照，然后每次翻页就是通过游标移动，获取下一页下一页这样子，性能会比上面说的那种分页性能也高很多很多

 

针对这个问题，你可以考虑用scroll来进行处理，scroll的原理实际上是保留一个数据快照，然后在一定时间内，你如果不断的滑动往后翻页的时候，类似于你现在在浏览微博，不断往下刷新翻页。那么就用scroll不断通过游标获取下一页数据，这个性能是很高的，比es实际翻页要好的多的多。

 

但是唯一的一点就是，这个适合于那种类似微博下拉翻页的，不能随意跳到任何一页的场景。同时这个scroll是要保留一段时间内的数据快照的，你需要确保用户不会持续不断翻页翻几个小时。

 

无论翻多少页，性能基本上都是毫秒级的

 

因为scroll api是只能一页一页往后翻的，是不能说，先进入第10页，然后去120页，回到58页，不能随意乱跳页。所以现在很多产品，都是不允许你随意翻页的，app，也有一些网站，做的就是你只能往下拉，一页一页的翻



**es生产集群的部署架构**

（1）es生产集群我们部署了5台机器，每台机器是6核64G的，集群总内存是320G

（2）我们es集群的日增量数据大概是2000万条，每天日增量数据大概是500MB，每月增量数据大概是6亿，15G。目前系统已经运行了几个月，现在es集群里数据总量大概是100G左右。

（3）目前线上有5个索引（这个结合你们自己业务来，看看自己有哪些数据可以放es的），每个索引的数据量大概是20G，所以这个数据量之内，我们每个索引分配的是8个shard，比默认的5个shard多了3个shard。

## 待补充

es底层的相关度评分算法（TF/IDF算法）

deep paging

上千万数据批处理

跨机房多集群同步

搜索效果优化