MySQL的基本存储结构是页(记录都存在页里边)：

 - **各个数据页可以组成一个双向链表**
 -   **每个数据页中的记录又可以组成一个单向链表**
       - 每个数据页都会为存储在它里边儿的记录生成一个页目录，在通过主键查找某条记录的时候可以在页目录中使用二分法快速定位到对应的槽，然后再遍历该槽对应分组中的记录即可快速找到指定的记录
       - 以其他列(非主键)作为搜索条件：只能从最小记录开始依次遍历单链表中的每条记录。


### 使用索引之后

索引做了些什么可以让我们查询加快速度呢？其实就是将无序的数据变成有序(相对)：



很明显的是：没有用索引我们是需要遍历双向链表来定位对应的页，现在通过 **“目录”** 就可以很快地定位到对应的页上了！（二分查找，时间复杂度近似为O(logn)）

其实底层结构就是B+树，B+树作为树的一种实现，能够让我们很快地查找出对应的记录。



> 以下内容整理自：《Java工程师修炼之道》


### 最左前缀原则

MySQL中的索引可以以一定顺序引用多列，这种索引叫作联合索引。如User表的name和city加联合索引就是(name,city)

```                                                                                       
select * from user where name=xx and city=xx ; ／／可以命中索引
select * from user where name=xx ; // 可以命中索引
select * from user where city=xx; // 无法命中索引            
```
这里需要注意的是，查询的时候如果两个条件都用上了，但是顺序不同，如 `city= xx and name ＝xx`，那么现在的查询引擎会自动优化为匹配联合索引的顺序，这样是能够命中索引的.

由于最左前缀原则，在创建联合索引时，索引字段的顺序需要考虑字段值去重之后的个数，较多的放前面。ORDERBY子句也遵循此规则。
​                         

- 如果是联合索引，那么key也由多个列组成，同时，索引只能用于查找key是否**存在（相等）**，遇到范围查询`(>、<、between、like`左匹配)等就**不能进一步匹配**了，后续退化为线性查找。
- 因此，**列的排列顺序决定了可命中索引的列数**。







hash索引

- 哈希索引也没办法利用索引完成**排序**
- 不支持**最左匹配原则**
- 在有大量重复键值情况下，哈希索引的效率也是极低的---->**哈希碰撞**问题。
- **不支持范围查询**









，考虑在`WHERE`和`ORDER BY`命令上涉及的列建立索引

`OR`改写成`IN`：`OR`的效率是n级别，`IN`的效率是log(n)级别，in的个数建议控制在200以内

应尽量避免在`WHERE`子句中对字段进行`NULL`值判断，否则将导致引擎放弃使用索引而进行全表扫描

- 避免`%xxx`式查询
- 少用`JOIN`

尽量避免在`WHERE`子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描



总体来讲，MyISAM适合`SELECT`密集型的表，而InnoDB适合`INSERT`和`UPDATE`密集型的表。在数据库做主从分离的情况下，经常选择MyISAM作为主库的存储引擎