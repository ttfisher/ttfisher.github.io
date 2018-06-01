---
title: Kafka入门系列（三）：Kafka常用shell探究
categories:
  - Big Data Component - Kafka
tags:
  - Kafka
  - Big Data
abbrlink: c0343485
date: 2018-05-30 16:57:51
---
【引言】本篇为Kafka入门系列的第三讲，经过第二讲的操作，我们已经搭建起了一个简单的Kafka集群了，那么怎么管理和使用这个集群呢？我们就先从Kafka提供的一些基础的shell脚本开始吧！
<div align=center><img src="/img/2018-05-30-02.png" width="330"/></div>
<!-- more -->

# Create：Topic的创建
```
./kafka-topics.sh --create --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 --replication-factor 3 --partitions 16 --topic kafka-test
```

# Create：Producer的创建
```
./kafka-console-producer.sh -broker-list 192.168.142.153:9092,192.168.142.156:9092,192.168.142.157:9092 -topic kafka-test
```

# Create：Consumer的创建
```
./kafka-console-consumer.sh --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 -topic kafka-test
```

# Read：Topic的查看
```
./kafka-topics.sh --list --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181
```

# Delete：Topic的数据清理（逻辑删除）

## 删除的操作步骤
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

## 补充说明
+ 此类操作的优势：不用删除topic
+ 此类操作的劣势：需要停服务

# Delete：Topic的数据清理（物理删除）

## 删除的操作步骤
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
## 假删除的补充操作
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

## 补充说明
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