---
title: Redis扫盲系列（一）：Redis基础入门
comments: true
categories:
  - 缓存&消息&数据库
  - Redis
tags:
  - Redis
  - Big Data
abbrlink: 23d37792
date: 2018-08-01 14:55:55
---
【引言】作为一个开发人员，稍微有一些开发经验的话，多多少少也会听说过redis的大名，你可以把它当做缓存，也可以把它作为消息队列，它可以做很多事情，而且它的性能还很强大，所以，这里开一个专题来聊聊关于Redis的点点滴滴吧！
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2018/2018-08-01-08.jpg" width="55%"/></div>
<!-- more -->

# Redis入门

## Redis是什么
&emsp;&emsp;[官方定义]Redis(Remote DIctionary Server) is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs and geospatial indexes with radius queries. Redis has built-in replication, Lua scripting, LRU eviction, transactions and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster.

## Redis支持的数据结构
+ String: 字符串
+ Hash: 散列
+ List: 列表
+ Set: 集合
+ Sorted Set: 有序集合

## Redis的几大特点
+ Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
+ Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
+ Redis支持数据的备份，即master-slave模式的数据备份。

## Redis的优势
+ 性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 。
+ 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
+ 原子 – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
+ 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。
+ Redis运行在内存中但是可以持久化到磁盘（RDB、AOF）

# Redis常用命令

## 基本操作

```
# 连接
$ redis-cli -h host -p port -a password

# 清库
$ redis 127.0.0.1:6379>flushdb

# 查看大小
$ redis 127.0.0.1:6379>dbsize

# 退出
$ redis 127.0.0.1:6379>quit

# 获取 redis 服务器的统计信息
$ redis 127.0.0.1:6379> INFO

# 常用命令使用模式
$ redis 127.0.0.1:6379> COMMAND KEY_NAME Option 
```

## Redis连接
```
- AUTH password 验证密码是否正确
- ECHO message 打印字符串
- PING 查看服务是否运行
- QUIT 关闭当前连接
- SELECT index 切换到指定的数据库

# 操作实例
redis 127.0.0.1:6379> AUTH "password"
OK
redis 127.0.0.1:6379> PING
PONG
```

## 脚本执行
```
# Redis 脚本使用 Lua 解释器来执行脚本。 Redis 2.6 版本通过内嵌支持 Lua 环境。执行脚本的常用命令为 EVAL。
- EVAL script numkeys key [key ...] arg [arg ...] 执行 Lua 脚本。
- EVALSHA sha1 numkeys key [key ...] arg [arg ...] 执行 Lua 脚本。
- SCRIPT EXISTS script [script ...] 查看指定的脚本是否已经被保存在缓存当中。
- SCRIPT FLUSH 从脚本缓存中移除所有脚本。
- SCRIPT KILL 杀死当前正在运行的 Lua 脚本。
- SCRIPT LOAD script 将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本。

# 操作实例
redis 127.0.0.1:6379> EVAL script numkeys key [key ...] arg [arg ...]
```

## key常用操作
```
- DEL key 该命令用于在 key 存在时删除 key。
- DUMP key 序列化给定 key ，并返回被序列化的值。
- EXISTS key 检查给定 key 是否存在。
- EXPIRE key seconds 为给定 key 设置过期时间。
- EXPIREAT key timestamp EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 
             不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。
- PEXPIRE key milliseconds 设置 key 的过期时间以毫秒计。
- PEXPIREAT key milliseconds-timestamp 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计
- KEYS pattern 查找所有符合给定模式( pattern)的 key 。
- MOVE key db 将当前数据库的 key 移动到给定的数据库 db 当中。
- PERSIST key 移除 key 的过期时间，key 将持久保持。
- PTTL key 以毫秒为单位返回 key 的剩余的过期时间。
- TTL key 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。
- RANDOMKEY 从当前数据库中随机返回一个 key 。
- RENAME key newkey 修改 key 的名称
- RENAMENX key newkey 仅当 newkey 不存在时，将 key 改名为 newkey 。
- TYPE key 返回 key 所储存的值的类型。
```

## string操作（前缀：无）
```
- SET key value 设置指定 key 的值
- GET key 获取指定 key 的值。
- GETRANGE key start end 返回 key 中字符串值的子字符
- GETSET key value 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。
- GETBIT key offset 对 key 所储存的字符串值，获取指定偏移量上的位(bit)。
- MGET key1 [key2..] 获取所有(一个或多个)给定 key 的值。
- SETBIT key offset value 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。
- SETEX key seconds value 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。
- SETNX key value 只有在 key 不存在时设置 key 的值。
- SETRANGE key offset value 用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。
- STRLEN key 返回 key 所储存的字符串值的长度。
- MSET key value [key value ...] 同时设置一个或多个 key-value 对。
- MSETNX key value [key value ...] 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
- PSETEX key milliseconds value 和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间。
- INCR key 将 key 中储存的数字值增一。
- INCRBY key increment 将 key 所储存的值加上给定的增量值（increment） 。
- INCRBYFLOAT key increment 将 key 所储存的值加上给定的浮点增量值（increment） 。
- DECR key 将 key 中储存的数字值减一。
- DECRBY key decrement key 所储存的值减去给定的减量值（decrement） 。
- APPEND key value 如果key已经存在并且是一个字符串，APPEND命令将指定的value追加到该 key 原来值（value）的末尾。
```

## hash操作（前缀：H）
```
- HDEL key field1 [field2] 删除一个或多个哈希表字段
- HEXISTS key field 查看哈希表 key 中，指定的字段是否存在。
- HGET key field 获取存储在哈希表中指定字段的值。
- HGETALL key 获取在哈希表中指定 key 的所有字段和值
- HINCRBY key field increment 为哈希表 key 中的指定字段的整数值加上增量 increment 。
- HINCRBYFLOAT key field increment 为哈希表 key 中的指定字段的浮点数值加上增量 increment 。
- HKEYS key 获取所有哈希表中的字段
- HLEN key 获取哈希表中字段的数量
- HMGET key field1 [field2] 获取所有给定字段的值
- HMSET key field1 value1 [field2 value2 ] 同时将多个 field-value (域-值)对设置到哈希表 key 中。
- HSET key field value 将哈希表 key 中的字段 field 的值设为 value 。
- HSETNX key field value 只有在字段 field 不存在时，设置哈希表字段的值。
- HVALS key 获取哈希表中所有值
- HSCAN key cursor [MATCH pattern] [COUNT count] 迭代哈希表中的键值对。

# 操作实例
127.0.0.1:6379>  HMSET runoobkey name "redis tutorial" description "redis basic commands for caching" likes 20 visitors 23000
OK
127.0.0.1:6379>  HGETALL runoobkey
1) "name"
2) "redis tutorial"
3) "description"
4) "redis basic commands for caching"
5) "likes"
6) "20"
7) "visitors"
8) "23000"
```

## list操作（前缀：L、R、BL、BR）
```
- BLPOP key1 [key2 ] timeout 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
- BRPOP key1 [key2 ] timeout 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
- BRPOPLPUSH source destination timeout 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
- LINDEX key index 通过索引获取列表中的元素
- LINSERT key BEFORE|AFTER pivot value 在列表的元素前或者后插入元素
- LLEN key 获取列表长度
- LPOP key 移出并获取列表的第一个元素
- LPUSH key value1 [value2] 将一个或多个值插入到列表头部
- LPUSHX key value 将一个值插入到已存在的列表头部
- LRANGE key start stop 获取列表指定范围内的元素
- LREM key count value 移除列表元素
- LSET key index value 通过索引设置列表元素的值
- LTRIM key start stop 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
- RPOP key 移除并获取列表最后一个元素
- RPOPLPUSH source destination 移除列表的最后一个元素，并将该元素添加到另一个列表并返回
- RPUSH key value1 [value2] 在列表中添加一个或多个值
- RPUSHX key value 为已存在的列表添加值

// 操作实例
redis 127.0.0.1:6379> LPUSH runoobkey redis
(integer) 1
redis 127.0.0.1:6379> LPUSH runoobkey mongodb
(integer) 2
redis 127.0.0.1:6379> LPUSH runoobkey mysql
(integer) 3
redis 127.0.0.1:6379> LRANGE runoobkey 0 10

1) "mysql"
2) "mongodb"
3) "redis"
```

## set操作（前缀：S）
```
- SADD key member1 [member2] 向集合添加一个或多个成员
- SCARD key 获取集合的成员数
- SDIFF key1 [key2] 返回给定所有集合的差集
- SDIFFSTORE destination key1 [key2] 返回给定所有集合的差集并存储在 destination 中
- SINTER key1 [key2] 返回给定所有集合的交集
- SINTERSTORE destination key1 [key2] 返回给定所有集合的交集并存储在 destination 中
- SISMEMBER key member 判断 member 元素是否是集合 key 的成员
- SMEMBERS key 返回集合中的所有成员
- SMOVE source destination member 将 member 元素从 source 集合移动到 destination 集合
- SPOP key 移除并返回集合中的一个随机元素
- SRANDMEMBER key [count] 返回集合中一个或多个随机数
- SREM key member1 [member2] 移除集合中一个或多个成员
- SUNION key1 [key2] 返回所有给定集合的并集
- SUNIONSTORE destination key1 [key2] 所有给定集合的并集存储在 destination 集合中
- SSCAN key cursor [MATCH pattern] [COUNT count] 迭代集合中的元素

# 操作实例
redis 127.0.0.1:6379> SADD runoobkey redis
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mongodb
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mysql
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mysql
(integer) 0
redis 127.0.0.1:6379> SMEMBERS runoobkey

1) "mysql"
2) "mongodb"
3) "redis"
```

## sorted set操作（前缀：S）
```
- ZADD key score1 member1 [score2 member2] 向有序集合添加一个或多个成员，或者更新已存在成员的分数
- ZCARD key 获取有序集合的成员数
- ZCOUNT key min max 计算在有序集合中指定区间分数的成员数
- ZINCRBY key increment member 有序集合中对指定成员的分数加上增量 increment
- ZINTERSTORE destination numkeys key [key ...] 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
- ZLEXCOUNT key min max 在有序集合中计算指定字典区间内成员数量
- ZRANGE key start stop [WITHSCORES] 通过索引区间返回有序集合成指定区间内的成员
- ZRANGEBYLEX key min max [LIMIT offset count] 通过字典区间返回有序集合的成员
- ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] 通过分数返回有序集合指定区间内的成员
- ZRANK key member 返回有序集合中指定成员的索引
- ZREM key member [member ...] 移除有序集合中的一个或多个成员
- ZREMRANGEBYLEX key min max 移除有序集合中给定的字典区间的所有成员
- ZREMRANGEBYRANK key start stop 移除有序集合中给定的排名区间的所有成员
- ZREMRANGEBYSCORE key min max 移除有序集合中给定的分数区间的所有成员
- ZREVRANGE key start stop [WITHSCORES] 返回有序集中指定区间内的成员，通过索引，分数从高到底
- ZREVRANGEBYSCORE key max min [WITHSCORES] 返回有序集中指定分数区间内的成员，分数从高到低排序
- ZREVRANK key member 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
- ZSCORE key member 返回有序集中，成员的分数值
- ZUNIONSTORE destination numkeys key [key ...] 计算给定的一个或多个有序集的并集，并存储在新的 key 中
- ZSCAN key cursor [MATCH pattern] [COUNT count] 迭代有序集合中的元素（包括元素成员和元素分值）

# 操作实例
redis 127.0.0.1:6379> ZADD runoobkey 1 redis
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 2 mongodb
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 3 mysql
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 3 mysql
(integer) 0
redis 127.0.0.1:6379> ZADD runoobkey 4 mysql
(integer) 0
redis 127.0.0.1:6379> ZRANGE runoobkey 0 10 WITHSCORES

1) "redis"
2) "1"
3) "mongodb"
4) "2"
5) "mysql"
6) "4"
```

# HyperLogLog
&emsp;&emsp;Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

## 什么是基数?
&emsp;&emsp;比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素的个数)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

## 四种基数统计方案
> 比如：在线人数统计

- 使用有序集合：这种方案能够同时储存在线的用户和用户上线时间，能够执行非常多的聚合计算，但是所消耗的内存也是非常可观的。
- 使用集合：这种方案能储存在线的用户，也能够执行一定的聚合计算，相对有序集合，所消耗的内存要小些，但是随着用户量的增多，消耗内存空间也处于增加状态。
- 使用hyperloglog：这种方案无论统计多少在线用户， 消耗的内存都是12k，但是只能给出在线用户的统计信息，无法获取准确的在线用户名单。
- 使用bitmap：这种方案还是比较好的，在尽可能节省内存空间情况下，记录在线用户的情况，而且能做一定的聚合运算。

# Redis的发布订阅
&emsp;&emsp;Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。Redis 客户端可以订阅任意数量的频道。

## 基本模型
&emsp;&emsp;下面两张图，第一张代表了订阅端对频道的订阅情况（即：3个客户端订阅了channel1这个频道的消息）；第二张图则表示当发送端发送消息时，消息通过该频道会推送到所有已订阅该频道的客户端（注：这里的模式和kafka稍有差异，kafka是客户端主动去pull消息的，而redis的客户端接收消息是由channel发起的push操作，本质不一样）。
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-08-02-02.jpg" width="40%">
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-08-02-01.jpg" width="40%">

## 命令说明
```
- PSUBSCRIBE pattern [pattern ...] 订阅一个或多个符合给定模式的频道。
- PUBSUB subcommand [argument [argument ...]] 查看订阅与发布系统状态。
- PUBLISH channel message 将信息发送到指定的频道。
- PUNSUBSCRIBE [pattern [pattern ...]] 退订所有给定模式的频道。
- SUBSCRIBE channel [channel ...] 订阅给定的一个或多个频道的信息。
- UNSUBSCRIBE [channel [channel ...]] 指退订给定的频道。
```

## 实例演示
```
# client订阅一个channel
redis 127.0.0.1:6379> SUBSCRIBE redisChat

Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redisChat"
3) (integer) 1

# server向该channel中publish一条消息
redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"

(integer) 1

redis 127.0.0.1:6379> PUBLISH redisChat "Learn redis by runoob.com"

(integer) 1

# 订阅者的客户端会显示如下消息
1) "message"
2) "redisChat"
3) "Redis is a great caching technique"

1) "message"
2) "redisChat"
3) "Learn redis by runoob.com"
```

# Redis的事务

## 事务特性
&emsp;&emsp;单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。
&emsp;&emsp;事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。redis的事务执行有以下特征：
- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

## 命令说明
```
- DISCARD 取消事务，放弃执行事务块内的所有命令。
- EXEC 执行所有事务块内的命令。
- MULTI 标记一个事务块的开始。
- UNWATCH 取消 WATCH 命令对所有 key 的监视。
- WATCH key [key ...] 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。
```

## 操作实例
```
# 如果在 set b bbb 处失败，set a 已成功不会回滚，set c 还会继续执行。
redis 127.0.0.1:7000> multi
OK
redis 127.0.0.1:7000> set a aaa
QUEUED
redis 127.0.0.1:7000> set b bbb
QUEUED
redis 127.0.0.1:7000> set c ccc
QUEUED
redis 127.0.0.1:7000> exec
1) OK
2) OK
3) OK
```

## 补充说明
**&emsp;&emsp;From redis docs on transactions: It's important to note that even when a command fails, all the other commands in the queue are processed – Redis will not stop the processing of commands.**


# Redis的安装

## 安装步骤
&emsp;&emsp;这个章节实际是验证OpenResty和Redis结合做反爬虫时发现没有
&emsp;&emsp;Redis因为没有多余的外部依赖，直接编译安装即可（但5.0以上新版本的貌似有些差异，暂不考虑），在这里我们用了4.0.2的发行版进行的编译安装，一般自测就直接使用redis-server启动了。
```
[root@localhost ~]# wget http://download.redis.io/releases/redis-4.0.2.tar.gz
......
[root@localhost ~]# tar -zxvf redis-4.0.2.tar.gz 
......
[root@localhost ~]# cd redis-4.0.2
[root@localhost redis-4.0.2]# make
cd src && make all
make[1]: Entering directory `/root/redis-4.0.2/src'
......
make[1]: Leaving directory `/root/redis-4.0.2/src'
[root@localhost redis-4.0.2]# make install
cd src && make install
make[1]: Entering directory `/root/redis-4.0.2/src'
......
    INSTALL install
make[1]: Leaving directory `/root/redis-4.0.2/src'
[root@localhost redis-4.0.2]# redis-server 
5367:C 18 Dec 21:15:26.420 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
5367:C 18 Dec 21:15:26.420 # Redis version=4.0.2, bits=64, commit=00000000, modified=0, pid=5367, just started
5367:C 18 Dec 21:15:26.420 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
5367:M 18 Dec 21:15:26.421 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 4.0.2 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 5367
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

5367:M 18 Dec 21:15:26.421 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
5367:M 18 Dec 21:15:26.421 # Server initialized
5367:M 18 Dec 21:15:26.422 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. ...
5367:M 18 Dec 21:15:26.422 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. ...
5367:M 18 Dec 21:15:26.422 * Ready to accept connections
^C5367:signal-handler (1545186523) Received SIGINT scheduling shutdown...
5367:M 18 Dec 21:28:43.443 # User requested shutdown...
5367:M 18 Dec 21:28:43.443 * Saving the final RDB snapshot before exiting.
5367:M 18 Dec 21:28:43.445 * DB saved on disk
5367:M 18 Dec 21:28:43.445 # Redis is now ready to exit, bye bye...
[root@localhost redis-4.0.2]# ^C
[root@localhost redis-4.0.2]# 
```
## 后台运行
&emsp;&emsp;若期望服务进程在后台运行，可通过redis.conf文件的daemonize配置项来解决，然后通过指定启动配置文件的方式来启动redis-server，这样就可以实现了：
```
[root@smart0002 redis-4.0.2]# redis-server /etc/redis.conf 
5430:C 19 Dec 11:05:18.557 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
5430:C 19 Dec 11:05:18.558 # Redis version=4.0.2, bits=64, commit=00000000, modified=0, pid=5430, just started
5430:C 19 Dec 11:05:18.558 # Configuration loaded
[root@smart0002 redis-4.0.2]# ps -ef|grep redis
root      5431     1  0 11:05 ?        00:00:00 redis-server 127.0.0.1:6379 
root      5446 29178  0 11:05 pts/6    00:00:00 grep redis
[root@smart0002 redis-4.0.2]# 
```

## 简单验证
&emsp;&emsp;在上面安装部署完成后，可通过自带的redis-cli连接进入缓存库（有点类似于mysql的命令操作模式），进入缓存库后可通过一系列的命令达到自己想要的目的（以下仅是很简单的示范）：
```
[root@localhost ~]# redis-cli -h localhost -p 6379
localhost:6379> dbsize
(integer) 4
localhost:6379> keys *
1) "timing_start_192.168.1.137"
2) "access_total_12/18/18"
3) "access_count_192.168.1.137"
4) "ban_time_192.168.1.137"
localhost:6379> get timing_start_192.168.1.137
"1545186401"
localhost:6379> 
```