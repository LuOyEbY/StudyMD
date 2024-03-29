# MYSQL避坑指南

##### 1.表属性设置为null会出现的问题

```java
1.列字段不指定NOT NULL 默认为NULL
2.NULL会占用存储空间
3.设定为NULL的列的长度不为0，为NULL。而空字符串的长度为0。
4.如果列字段的值为NULL，则需要使用IS 或者 IS NOT进行判断。
5.允许给NULL列添加索引，但是实际查询过程中只有IS和IS NOT才会使用索引。
6.NULL进行计算，与任何值进行计算最终都会得到NULL。
7.允许为NULL的列进行count()聚合函数，返回的是排除null之后的聚合。
8.允许为NULL的列进行排序。ASC NULL值排前边；DESC NULL值排后边。
```

##### 结论：任何场景下不要将列的默认值设置为null

```java
1.使用特殊值去填充NULL,ex:空字符串或者数字0.
2.对于已经存在数据的表，先填充特殊值到NULL列，然后再去修改表结构。
```



##### 2.不要随意设置数据类型，不给未来留隐患

```java
1.指定主键，主键不具有任何业务含义，只是一个唯一的自增整数值。
2.如果不指定，MYSQL会默认添加隐式主键。
3.数据类型的取值范围：
  a.字符串：char、varchar、[tinytext、text、mediumtext、longtext]
  b.日期/时间：date、time、datetime、timestamp
  c.数值：tinyint、int、bigint、float、double、decimal
  d.二进制：tityblob、blob、mediumblob、longblob
4.枚举ENUM（不要使用）
```

##### 结论：

```java
1.使用存储所需要的最小数据类型
2.选择简单的数据类型
3.存储小数直接用decimal
4.尽量避免使用text和blob
```

##### 3.MYSQL索引加的不好，效果会适得其反

```java
1.索引加的正确，但是没有写出使用索引的查询语句
  a.字符串类型在查询是没有使用引号，不会使用表索引。
  b.where条件左边的字段参与了函数或者数学运算，不会使用表索引。
  c.联合索引最左前缀顺序不匹配，不会使用表索引。
2.索引增加的不正确，或者冗余
  a.不再使用的索引没有及时删除：空间浪费、插入删除更新性能受到影响、MYSQL维护索引也是需要消耗资源的。
  b.索引选择性太低，索引列的意义不大。索引选择性=不重复的索引值/表记录数
  c.列值多长，可以选择部分前缀作为索引（区分度高的情况下），而不是整列加上索引。
```

##### 结论：

```java
1.表记录较少，全表扫描效率更高，1000行为界限
2.存在联合索引的情况下，再去对前缀部分加索引（已经覆盖了单列或者多列索引）
3.一张表中建立的索引过多（超过5个），应该是根据业务需求去分析创建，并不是越多越好，太多会浪费空间，影响额外的查询效率。
```

##### 4.MYSQL为什么会莫名其妙的断开连接

```java
1.MYSQL的默认连接超时时间是8个小时
2.autoReconnect参数的副作用：
  a.原有连接上的事务将会被回滚，事务的提交模式会丢失。
  b.原有连接中持有的表的锁将会全部释放。
  c.原有连接关联的会话session将会丢失，重新恢复的连接关联的将会是一个全新的session。
  d.原有连接中定义的用户变量将会丢失。
  e.原有连接中定义的预编译SQL将会丢失。
  f.原有连接失效，新的连接恢复之后，MYSQL将使用新的记录行来存储连接中的性能数据。
```

##### 结论：

```java
1.修改MYSQL配置，避免断开连接
	a.MYSQL中连接的两个核心变量：interactive_timeout,wait_timeout
2.数据库连接池HiKariCP
  a.maximum-pool-size:最大连接数据。超过这个数目，新的数据库访问线程会被阻塞。推荐：（（cpu_core * 2））+ HD_COUNT
  b.minimum-idle:最小连接数目
  c.max-lifetime:最大的连接生命时间，用来设置一个connection在连接池中的存活时间。
  d.idle-timeout:一个连接idle状态的最大时长。超时则被释放掉。生效条件：minimun-idle < maximum-pool-size && idle-timeout > 0
```

##### 5.事务处理出错？可能是锁用的不对

```java
1.数据库锁的分类
  a.按照锁数据的粒度分为：行级锁和表级锁
  b.按照数据的锁定方式去分：乐观锁和悲观锁（排它锁和共享锁）
2.行锁的并发性能远高于表锁，但是小心粒度升级
  a.InnoDB只有在通过索引条件检索数据时使用行级锁，否则使用表锁
  b.对于UPDATE,DELETE和INSERT语句，InnoDB会自动给涉及数据集加排它锁；对于普通SELECT语句，InnoDB不会加任何锁
```

##### 6.你写的SQL很慢，怎样做优化呢？（慢查询）

```java
1.配置MYSQL慢查询的参数
  a.slow_query_log:标记慢查询日志是否开启的参数，默认是OFF，既不开启
  b.slow_query_log_file:标记存放慢查询日志文件的完整路径（mysql要有写入此路径文件的权限）
  c.long_query_time:控制慢查询的时间阈值参数
  d.log_queries_not_using_indexes:标识是否记录未使用索引的查询，默认是关闭。
2.使用mysqldumpslow工具解读慢查询日志
   mysqdumpslow -s t -t 10 -g "group by" /tmp/slow_quer.log
  a: -s 参数选择排序方式，记录次数c;时间t;查询时间l;返回记录数r;前边加a标识相应的倒序。
  b: -t 返回Top N 个数据。
  c: -g 正则匹配模式，sql语句中的关键字，大小写不敏感。
3.使用explain/desc分析执行的SQL语句
  a:select_type:查询中每个select子句的类型
  b:possible_keys:可能使用的索引
  c:keys:实际使用的索引
  d:key_len:索引中使用的字节数
  e:rows：估算需要读取的数据行
  f:filtered:满足查询的记录数量的比例
4.慢查询问题总结
  a:select太多数据行、数据列（大数据量查询）
  b:没有定义合适的索引
  c:没有按照索引定义的顺序查询
  d:定义了索引，但是mysql并没有选择
  e:使用了复杂查询
```

##### 7.数据量逐渐增大，才考虑分库分表可行吗？

```java
1.业务发展过程中遇到的问题
  a:随着业务发展，数据量急剧膨胀，查询，迁移会变得非常麻烦，最经典的例子是用户表和订单表
  b:表设计的不合理，在使用过程中发现问题（列太多，且99%的查询大部分的列都不会被用到）
2.分库分表的策略
  a:垂直切分：可以同时作用于库和表
  b:水平切分：只适用于数据表（按照时间区间或者自增ID来切分）（哈希取模切分）
3.垂直切分的优缺点
  a:消除库中存在的业务表耦合，是数据表之间的关系更加的清晰（优点）
  b:将数据库的连接资源、单机硬件资源隔离开，更利于业务的扩展（优点）
  c:表与表之间很难做到完整的JION，只能通过多次查询的方式聚合数据（缺点）
  d:查询多个表会将单表事务升级为分布式事务，实现难度大大增加（缺点）
  e:扔然可能会存在单表数据量过大的问题（缺点）
4.水平切分的优缺点
  a:不会存在单表数据量过大的问题，改动成本低
  b:数据记录跨多张表依然会有分布式事务的问题
  c:表之间的JOIN由于数据量不全会比较麻烦
5.分库分表引发的问题
  a:全局主键问题（使用UUID,但是数据量比较长）（额外自增表，容错率较低）（分布式全局唯一ID生成算法：twiter和美团的算法）
  b:事务一致性问题：强一致性和最终一致性（XA协议）（2PC两阶段提交）（3PC三阶段提交）
  c:关联查询JOIN问题（字段冗余设计）（数据组装--后端）（拆分查询--前端）
  
```

