
#初识redis
##数据结构简介
结构类型|结构存储的值|结构的读写能力
-------|------------|------------
STRING|可以是字符串、整数或则浮点数|对整个字符串或者字符串的其中一部分执行操作；对整数和浮点数执行自增或则自减操作
LIST|一个链表，链表上的每个节点都包含了一个字符串|从链表的两端推入或则弹出元素，根据偏移量对链表进行修剪；读取单个或则多个元素；根据值查找或则移除元素
SET|包含字符串的无序收集器，并且包含的每个字符串都是独一无二的、各不相同的|添加、获取、移除单个元素；检查一个元素是否存在于集合中；计算交集、并级、差集；从集合里面随机获取元素
HASH|包含键值对的无序列表|添加、获取、移除单个键值对；获取所有键值对
ZSET(有序集合)|字符串成员（member）与浮点数分值（score）之间的有序映射，元素的排列顺序由分值大小决定|添加、获取、删除单个元素；根据分值范围（range）或则成员来获取元素


### 字符串
>命令：get、set、del;还提供对存储的数值进行自增或则自减操作

```shell
127.0.0.1:6379> set name xq
OK
127.0.0.1:6379> set name xl
OK
127.0.0.1:6379> get name
"xl"
127.0.0.1:6379> del name
(integer) 1
127.0.0.1:6379> get name
(nil)

```

### 列表
链表（linked-list）: list-key --> list
1. key --> list；比如：RPUSH name xq;RPUSH name xl; xq和小丽都属于name这个key中
2. 值有序，可重复

命令|行为
-|-
RPUSH|将给定值推入列表的右端
LRANGE|获取列表在给定范围上的所有值
LINDEX|获取列表在给定位置上的单个元素
LPOP|从列表的左端弹出一个值，并返回被弹出的值

```shell
127.0.0.1:6379> rpush names xq
(integer) 1
127.0.0.1:6379> rpush names xl
(integer) 2
127.0.0.1:6379> rpush names xl2
(integer) 3
127.0.0.1:6379> rpush names xl1
(integer) 4
127.0.0.1:6379> lrange names
(error) ERR wrong number of arguments for 'lrange' command
127.0.0.1:6379> lrange names 0 -1
1) "xq"
2) "xl"
3) "xl2"
4) "xl1"
127.0.0.1:6379> lpop names
"xq"
127.0.0.1:6379> lrange names 0 -1
1) "xl"
2) "xl2"
3) "xl1"
127.0.0.1:6379> lindex names 2
"xl1"
127.0.0.1:6379> lrange names 0 -1
1) "xl"
2) "xl2"
3) "xl1"
```

###集合
set-key --> set

1. 与列表差不多的数据结构，都是 key ---> set 方式；
2. 无序，set中的值不可重复

命令|行为
-|-
SADD|将给定元素添加到集合
SMEMBERS|返回集合包含的所有元素
SISMEMBER|检查给定元素是否存在于集合中
SREM|如果给定的元素存在于集合中，那么移除这个元素

```shell
增加几个名字到key=names1 的集合中
127.0.0.1:6379> sadd names1 xq
(integer) 1
127.0.0.1:6379> sadd names1 xl
(integer) 1
127.0.0.1:6379> sadd names1 xl
(integer) 0

返回所有元素
127.0.0.1:6379> smembers names1
1) "xq"
2) "xl"

是否存在xq这个元素
127.0.0.1:6379> sismember names1 xq
(integer) 1
127.0.0.1:6379> sismember names1 xq1
(integer) 0

存在xq这个元素就删除它
127.0.0.1:6379> srem names1 xq
(integer) 1

127.0.0.1:6379> smembers names1
1) "xl"

```




###散列
hash-key  --> hash

1. 和字符串一样，可以存储字符串和数字，支持自增和自减操作
2. 键key，无序

命令|行为
-|-
HSET|在散列里面关联起给定的键值对
HGET|获取指定散列的值
HGETALL|获取散列包含的所有键值对
HDEL|如果给定键存在于散列里面，那么移除这个键

```Shell
往散列中存入 KEY=class,key=c1,val=xq 的数据
127.0.0.1:6379> hset class c1 xq
(integer) 1
这里其实是对上一条数据的更新，因为key是hash的，唯一的
127.0.0.1:6379> hset class c1 xl
(integer) 0
127.0.0.1:6379> hset class c2 xq
(integer) 1
127.0.0.1:6379> hset class c2 lp
(integer) 0
上面4条语句，其实只插入了两条数据。有两条命令是对上一条的更新

下面的获取所有的数据，在命令行中，显示有4条，其实奇数行是key，偶数行是val
127.0.0.1:6379> hgetall class
1) "c1"
2) "xl"
3) "c2"
4) "lp"

127.0.0.1:6379> hget class c1
"xl"
这里可以证明上面的4行数据，是key和val组成的
127.0.0.1:6379> hget class xl
(nil)

删除
127.0.0.1:6379> hdel class c1
(integer) 1
127.0.0.1:6379> hgetall class
1) "c2"
2) "lp"

```

###有序集合
zset-key --> zset
1. 键称为成员（member）
2. 值称为分值（score）
3. 有序集合是redis中唯一一个既可以根据成员访问元素，又可以根据分值以及分值的排列顺序来访问元素的结构

命令|行为
-|-
ZADD|将一个带有给定分值的成员添加到有序集合里面
ZRANGE|根据元素在有序排列中所处的位置，从有序集合里面获取多个元素
ZRANGEBYSCORE|获取有序集合在给定分值范围内的所有元素
ZREM|如果给定成员存在于有序集合，那么移除这个成员

```bash
往 KEY=znames，分值=整数浮点数如728，成员=m1 的有序集合插入一条数据
127.0.0.1:6379> zadd znames 728 m1
(integer) 1
127.0.0.1:6379> zadd znames 600 m2
(integer) 1
127.0.0.1:6379> zadd znames 900 m0
(integer) 1

获取所有的成员
127.0.0.1:6379> zrange znames 0 -1
1) "m2"
2) "m1"
3) "m0"

获取所有的成员，显示分值
127.0.0.1:6379> zrange znames 0 -1 withscores
1) "m2"
2) "600"
3) "m1"
4) "728"
5) "m0"
6) "900"

语法一定要对，不然就提示无效的哦
127.0.0.1:6379> zadd znames m3 800
(error) ERR value is not a valid float

指定获取分值为600 - 800 的成员
127.0.0.1:6379> zrangebyscore znames 600 800
1) "m2"
2) "m1"

删除成员m0
127.0.0.1:6379> zrem znames m0
(integer) 1
127.0.0.1:6379> zrange znames 0 -1 withscores
1) "m2"
2) "600"
3) "m1"
4) "728"

```