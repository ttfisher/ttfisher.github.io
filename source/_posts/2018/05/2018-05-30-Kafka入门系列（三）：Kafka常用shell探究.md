---
title: Kafka入门系列（三）：Kafka常用shell探究
categories:
  - 消息和缓存系列之Kafka
tags:
  - Kafka
  - Big Data
abbrlink: 409c4b9a
date: 2018-05-30 16:57:51
---
【引言】本篇为Kafka入门系列的第三讲，经过第二讲的操作，我们已经搭建起了一个简单的Kafka集群了，那么怎么管理和使用这个集群呢？我们就先从Kafka提供的一些基础的shell脚本开始吧！
<div align=center><img src="/img/2018/2018-05-30-02.jpg" width="330"/></div>
<!-- more -->

# 服务管理
```
# 启动zookeeper
./zookeeper-server-start.sh config/zookeeper.properties &

# 启动kafka
./kafka-server-start.sh ../config/server.properties &

# 停止kafka
./kafka-server-stop.sh

# 停止zookeeper
./zookeeper-server-stop.sh

# 下线broker  
./kafka-run-class.sh kafka.admin.ShutdownBroker --zookeeper 192.168.142.153:2181 --broker #brokerId# --num.retries 3 --retry.interval.ms 60
shutdown broker

# zookeeper如何连接
./zkCli.sh -server host:port cmd args

# 查询偏移量（zookeeper）
get /consumers/consumer-group/offsets/TOPIC_NAME/0

# 设置偏移量（zookeeper）
set /consumers/consumer-group/offsets/TOPIC_NAME/0 10086
```

# Topic的管理

## 常用的操作Shell
```
# 普通Topic创建
./kafka-topics.sh --create --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 --replication-factor 3 --partitions 16 --topic kafka-test

# 带特殊参数的Topic
./kafka-topics.sh --create --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 --replication-factor 3 --partitions 16 --topic kafka-test3 --config cleanup.policy=delete --config retention.bytes=11 --config retention.ms=60

# 为Topic增加Partition
./bin/kafka-topics.sh -zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 -alter -partitions 20 -topic kafka-test3

# 为Topic增加副本
待补充

# Topic的查看（列表）
./kafka-topics.sh --list --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181

# 实例
[root@b3e6614d0345 bin]# ./kafka-topics.sh --list --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181
__consumer_offsets
kafka-test2
kafka-test3
[root@b3e6614d0345 bin]# 

# Topic的查看（单个）
./kafka-topics.sh --describe --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 --topic kafka-test3

# 实例
[root@b3e6614d0345 bin]# ./kafka-topics.sh --describe --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 --topic kafka-test3
Topic:kafka-test3       PartitionCount:16       ReplicationFactor:3     Configs:retention.ms=60,cleanup.policy=delete,retention.bytes=1
        Topic: kafka-test3      Partition: 0    Leader: 1       Replicas: 1,3,2 Isr: 1,3,2
        Topic: kafka-test3      Partition: 1    Leader: 2       Replicas: 2,1,3 Isr: 2,1,3
        Topic: kafka-test3      Partition: 2    Leader: 3       Replicas: 3,2,1 Isr: 3,2,1
        Topic: kafka-test3      Partition: 3    Leader: 1       Replicas: 1,2,3 Isr: 1,2,3
        Topic: kafka-test3      Partition: 4    Leader: 2       Replicas: 2,3,1 Isr: 2,3,1
        Topic: kafka-test3      Partition: 5    Leader: 3       Replicas: 3,1,2 Isr: 3,1,2
        Topic: kafka-test3      Partition: 6    Leader: 1       Replicas: 1,3,2 Isr: 1,3,2
        Topic: kafka-test3      Partition: 7    Leader: 2       Replicas: 2,1,3 Isr: 2,1,3
        Topic: kafka-test3      Partition: 8    Leader: 3       Replicas: 3,2,1 Isr: 3,2,1
        Topic: kafka-test3      Partition: 9    Leader: 1       Replicas: 1,2,3 Isr: 1,2,3
        Topic: kafka-test3      Partition: 10   Leader: 2       Replicas: 2,3,1 Isr: 2,3,1
        Topic: kafka-test3      Partition: 11   Leader: 3       Replicas: 3,1,2 Isr: 3,1,2
        Topic: kafka-test3      Partition: 12   Leader: 1       Replicas: 1,3,2 Isr: 1,3,2
        Topic: kafka-test3      Partition: 13   Leader: 2       Replicas: 2,1,3 Isr: 2,1,3
        Topic: kafka-test3      Partition: 14   Leader: 3       Replicas: 3,2,1 Isr: 3,2,1
        Topic: kafka-test3      Partition: 15   Leader: 1       Replicas: 1,2,3 Isr: 1,2,3
[root@b3e6614d0345 bin]# 

# Topic的两种删除方法
./kafka-run-class.sh kafka.admin.DeleteTopicCommand --zookeeper 192.168.142.153:2181 --topic kafka-test3 
./kafka-topics.sh --zookeeper 192.168.142.153:2181 --delete --topic kafka-test3  

# 查询最大的offset
./kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 192.168.142.153:9092,192.168.142.156:9092,192.168.142.157:9092 -topic kafka-test3 --time -1

```

## Topic的级别配置
&emsp;&emsp;配置topic级别参数时，相同（参数）属性topic级别会覆盖全局的（也就是server.properties文件内的配置），否则默认为全局配置属性值。创建topic参数可以设置一个或多个--config "Property(属性)"；但要保证每个配置项前的--config不可缺少。以下提供几个实例参考：
```
# 创建topic时配置参数
./kafka-topics.sh --zookeeper 192.168.142.153:2181/kafka01 --create --topic test-topic --partitions 1   --replication-factor 1 --config max.message.bytes=64000 --config flush.messages=1

# 修改topic时配置参数，会覆盖已经有topic参数，下面例子修改"my-topic"的max message属性
./kafka-topics.sh --zookeeper 192.168.142.153:2181/kafka01 --alter --topic test-topic --config max.message.bytes=128000

# 删除topic级别配置参数
./kafka-topics.sh --zookeeper 192.168.142.153:2181/kafka01 --alter --topic test-topic --delete-config max.message.bytes

# 【注】：这里Zookeeper指定了目录了，所以这里创建的Topic元数据都在指定的目录下，所以test_topic在Zookeeper下的存储内容如下
{
    "version": 1,
    "config": {
        "max.message.bytes": "12800000",
        "flush.messages": "1000"
    }
}
```

## Topic级别列表
&emsp;&emsp;以下topic级别列表（来源于网络，表示一下感谢！O(∩_∩)O），其中kafak server中默认配置为下表“Server Default Property”列，当需要在创建topic时设置topic级别则按照“Property(属性)”列的名称设置即可。

|Property(属性)|Default(默认值)|Server Default Property(server.properties)|说明(解释)|
|:--:|:--:|:--:|:--:|
|cleanup.policy|delete|log.cleanup.policy|日志清理策略选择有：delete和compact主要针对过期数据的处理，或是日志文件达到限制的额度，会被 topic创建时的指定参数覆盖|
|delete.retention.ms|86400000 (24 hours)|log.cleaner.delete.retention.ms|对于压缩的日志保留的最长时间，也是客户端消费消息的最长时间，同log.retention.minutes的区别在于一个控制未压缩数据，一个控制压缩后的数据。会被topic创建时的指定参数覆盖|
|flush.messages|None|log.flush.interval.messages|log文件”sync”到磁盘之前累积的消息条数,因为磁盘IO操作是一个慢操作,但又是一个”数据可靠性"的必要手段,所以此参数的设置,需要在"数据可靠性"与"性能"之间做必要的权衡.如果此值过大,将会导致每次"fsync"的时间较长(IO阻塞),如果此值过小,将会导致"fsync"的次数较多,这也意味着整体的client请求有一定的延迟.物理server故障,将会导致没有fsync的消息丢失.|
|flush.ms|None|log.flush.interval.ms|仅仅通过interval来控制消息的磁盘写入时机,是不足的.此参数用于控制"fsync"的时间间隔,如果消息量始终没有达到阀值,但是离上一次磁盘同步的时间间隔达到阀值,也将触发.|
|index.interval.bytes|4096|log.index.interval.bytes|当执行一个fetch操作后，需要一定的空间来扫描最近的offset大小，设置越大，代表扫描速度越快，但是也更好内存，一般情况下不需要搭理这个参数|
|message.max.bytes|1,000,000|message.max.bytes|表示消息的最大大小，单位是字节|
|min.cleanable.dirty.ratio|0.5|log.cleaner.min.cleanable.ratio|日志清理的频率控制，越大意味着更高效的清理，同时会存在一些空间上的浪费，会被topic创建时的指定参数覆盖|
|retention.bytes|None|log.retention.bytes|topic每个分区的最大文件大小，一个topic的大小限制 = 分区数*log.retention.bytes。-1没有大小限log.retention.bytes和log.retention.minutes任意一个达到要求，都会执行删除，会被topic创建时的指定参数覆盖|
|retention.ms|None|log.retention.minutes|数据存储的最大时间超过这个时间会根据log.cleanup.policy设置的策略处理数据，也就是消费端能够多久去消费数据，log.retention.bytes和log.retention.minutes达到要求，都会执行删除，会被topic创建时的指定参数覆盖|
|segment.bytes|1 GB|log.segment.bytes|topic的分区是以一堆segment文件存储的，这个控制每个segment的大小，会被topic创建时的指定参数覆盖|
|segment.index.bytes|10 MB|log.index.size.max.bytes|对于segment日志的索引文件大小限制，会被topic创建时的指定参数覆盖|
|log.roll.hours|7 days|log.roll.hours|这个参数会在日志segment没有达到log.segment.bytes设置的大小，也会强制新建一个segment会被 topic创建时的指定参数覆盖|

## 数据清理方式一

### 删除的操作步骤
+ 第一步：需要停止当前的Kafka服务
+ 第二步：删除Kafka消息日志文件目录
 + 也就是log.dirs配置的目录（grep log.dirs /kafka/kafka_2.10-0.9.0.0/config/server.properties）
 + 删除的目录是以需要删除的topic为前缀命名的
 + 目录的数量不定，是跟kafka的配置中的partition来进行分割的
+ 第三步：修改Zookeeper的topic偏移量（offset）
 + 进入Zookeeper的安装目录下，运行./zkCli.sh -server (多个server中间以逗号分割)，如果不带server默认只修改本机数据
 + 在Zookeeper上运行ls /consumers/对应的分组/offset/对应的topic，就可以看到此topic下的所有分区了
 + 通过get /consumers/对应的分组/offset/对应的topic/对应的分区号，可以查询到该分区上记录的offset
 + 通过set /consumers/对应的分组/offset/对应的topic/对应的分区号 修改后的值（一般为0），即可完成对offset的修改（应该是需要对每个分区都进行offset归零操作的）
```
[zk: ZK.Servers(CONNECTED) 12] ls /consumers/console-consumer-73536/offsets/kafka-test2
[15, 13, 14, 11, 12, 3, 2, 1, 10, 0, 7, 6, 5, 4, 9, 8]
[zk: ZK.Servers(CONNECTED) 25] get /consumers/console-consumer-73536/offsets/kafka-test2/5
1
cZxid = 0x20000017b
ctime = Thu May 31 09:34:08 UTC 2018
mZxid = 0x2000001a4
mtime = Thu May 31 09:38:08 UTC 2018
pZxid = 0x20000017b
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0
[zk: ZK.Servers(CONNECTED) 26] set /consumers/console-consumer-73536/offsets/kafka-test2/5 0
cZxid = 0x20000017b
ctime = Thu May 31 09:34:08 UTC 2018
mZxid = 0x2000001b6
mtime = Thu May 31 09:41:24 UTC 2018
pZxid = 0x20000017b
cversion = 0
dataVersion = 3
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0
[zk: ZK.Servers(CONNECTED) 27] get /consumers/console-consumer-73536/offsets/kafka-test2/5  
0
cZxid = 0x20000017b
ctime = Thu May 31 09:34:08 UTC 2018
mZxid = 0x2000001b6
mtime = Thu May 31 09:41:24 UTC 2018
pZxid = 0x20000017b
cversion = 0
dataVersion = 3
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0
[zk: ZK.Servers(CONNECTED) 28] 
```
+ 第四步：重启Kafka服务

### 补充说明
+ 此类操作的优势：不用删除topic
+ 此类操作的劣势：需要停服务

## 数据清理方式二

### 删除的操作步骤
+ 第一步：清理kafka数据目录（也就是log.dirs配置的目录，删除以需要删除的topic为前缀的所有目录，目录数是跟具体的partition配置一致的）
+ 第二步：通过kafka-topics.sh --delete删除kafka topic；
 + 如果当前topic没有使用过即没有传输过信息：可彻底删除
 + 如果当前topic有使用过即有过传输过信息：并不会真正删除topic，只是把这个topic标记为删除（marked for deletion）。要彻底把这种topic删除必须把kafka中与当前topic相关的数据目录和zookeeper与当前topic相关的路径一并删除。
+ 第三步：删除zookeeper相关数据（多个目录）
 + 一般情况下需要删除的目录：rmr /admin/delete_topics/kafka-test、rmr /brokers/topics/kafka-test
 + 正常情况是不需要进行的两个操作：rmr /consumers/kafka-test-group、rmr /config/topics/kafka-test
+ 第四步：确认topic是否还存在（查看topic列表即可）
```
[root@1bc72703f1e9 ~]# grep log.dirs /kafka/kafka_2.10-0.9.0.0/config/server.properties 
log.dirs=/kafka/kafka-logs
[root@1bc72703f1e9 ~]# rm -rf /kafka/kafka-logs/kafka-test-*
[root@1bc72703f1e9 ~]# /kafka/kafka_2.10-0.9.0.0/bin/kafka-topics.sh --delete --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 --topic kafka-test
Topic kafka-test is marked for deletion.
Note: This will have no impact if delete.topic.enable is not set to true.
[root@1bc72703f1e9 ~]# cd /zookeeper/zookeeper-3.4.6/bin
[root@1bc72703f1e9 bin]# ./zkCli.sh -server 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181

Connecting to 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181
......
WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181(CONNECTED) 5] ls /
[consumers, config, controller, isr_change_notification, admin, brokers, zookeeper, controller_epoch]
[zk: 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181(CONNECTED) 6] rmr /consumers/kafka-test-group
Node does not exist: /consumers/kafka-test-group
[zk: 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181(CONNECTED) 7] rmr /config/topics/kafka-test
Node does not exist: /config/topics/kafka-test
[zk: 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181(CONNECTED) 8] rmr /brokers/topics/kafka-test
Node does not exist: /brokers/topics/kafka-test
[zk: 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181(CONNECTED) 9] rmr /admin/delete_topics/kafka-test
Node does not exist: /admin/delete_topics/kafka-test
[zk: 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181(CONNECTED) 10] 
[root@1bc72703f1e9 bin]# /kafka/kafka_2.10-0.9.0.0/bin/kafka-topics.sh --list --zookeeper 92.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181
kafka-test2
[root@1bc72703f1e9 bin]# 
```
### 假删除的补充操作
&emsp;&emsp;如果kafka启动时加载的配置文件server.properties没有配置**<font color=red>delete.topic.enable = true</font>**，那么此时的删除并不是真正的删除。而只是把topic标记为：marked for deletion,此时就需要执行如下操作：
+ 连接Zookeeper：./zkCli.sh -server 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181
+ 删除相关信息：进入/admin/delete_topics目录下，找到删除的topic,删除对应的信息（如果删除了此处的topic，那么marked for deletion标记消失）
```
[root@1bc72703f1e9 bin]# ./zkCli.sh -server 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181

Connecting to 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181
......
WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181(CONNECTED) 0] ls /admin/delete_topics
[]
[zk: 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181(CONNECTED) 1] rmr XXX
```

### 补充说明
&emsp;&emsp;**<font color=red>使用kafka delete时，如果需要被删除topic 此时正在被程序 produce和consume，则这些生产和消费程序需要停止，因为不停止的话该topic的offset信息一致会在broker持续更新。</font>**
&emsp;&emsp;**<font color=red>需要设置 auto.create.topics.enable = false，默认设置为true。如果设置为true，则produce或者fetch 不存在的topic也会自动创建这个topic。这样会给删除topic带来很多意想不到的问题。</font>**
&emsp;&emsp;**<font color=red>通常情况下，创建了新的topic之后，在Zookeeper下对应的目录都有相应的文件的，所以在删除topic的时候需要对这些目录进行删除，否则会出现数据一致性的问题。</font>**
```
[zk: 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181(CONNECTED) 13] ls /config/topics
[]
[zk: 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181(CONNECTED) 14] ls /config/topics
[kafka-test2]
[zk: 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181(CONNECTED) 15] 
```

# Producer的管理

## 常用操作
```
# 普通的Producer
./kafka-console-producer.sh -broker-list 192.168.142.153:9092,192.168.142.156:9092,192.168.142.157:9092 -topic kafka-test
```

# Consumer的管理

## 常用操作
```
# 普通的Consumer
./kafka-console-consumer.sh --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 -topic kafka-test

# 从起始位置开始消费的Consumer
./kafka-console-consumer.sh --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 --from-beginning  -topic kafka-test 

# 查看consumer组内消费的offset  
./kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --zookeeper 192.168.142.153:2181 --group tt --topic kafka-test3
```