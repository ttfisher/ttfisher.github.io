---
title: Kafka入门系列（二）：常用shell和配置文件
categories:
  - 消息和缓存系列之Kafka
tags:
  - Kafka
  - Big Data
abbrlink: 409c4b9a
date: 2018-05-30 16:57:51
---
【引言】本篇为Kafka入门系列的第三讲，经过第二讲的操作，我们已经搭建起了一个简单的Kafka集群了，那么怎么管理和使用这个集群呢？我们就先从Kafka提供的一些基础的shell脚本开始吧！
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2018/2018-05-30-02.jpg" width="330"/></div>
<!-- more -->

# 常用shell

## 服务管理
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

## Topic的管理

### 常用的操作Shell
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

### Topic的级别配置
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

### Topic级别列表
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

### 数据清理方式一

#### 删除的操作步骤
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

#### 补充说明
+ 此类操作的优势：不用删除topic
+ 此类操作的劣势：需要停服务

### 数据清理方式二

#### 删除的操作步骤
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
#### 假删除的补充操作
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

#### 补充说明
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

## Producer的管理

### 常用操作
```
# 普通的Producer
./kafka-console-producer.sh -broker-list 192.168.142.153:9092,192.168.142.156:9092,192.168.142.157:9092 -topic kafka-test
```

## Consumer的管理

### 常用操作
```
# 普通的Consumer
./kafka-console-consumer.sh --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 -topic kafka-test

# 从起始位置开始消费的Consumer
./kafka-console-consumer.sh --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 --from-beginning  -topic kafka-test 

# 查看consumer组内消费的offset  
./kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --zookeeper 192.168.142.153:2181 --group tt --topic kafka-test3
```

# 配置文件详解

## 默认配置项（kafka_2.10-0.9.0.0）
```
[root@63c3077b732d kafka-test3-1]# cat /kafka/kafka_2.10-0.9.0.0/config/server.properties
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
# 
#    http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# see kafka.server.KafkaConfig for additional details and defaults

############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
# 在集群中的节点id，要求：必须为正整数，在整个集群中不得重复
broker.id=3

############################# Socket Server Settings #############################

# 监听列表，broker对外提供服务时绑定的IP和端口
listeners=PLAINTEXT://:9092

# The port the socket server listens on
# 服务监听端口，默认值是9092，一般不做改动
port=9092

# Hostname the broker will bind to. If not set, the server will bind to all interfaces
# 服务监听地址，一般配置为本机的hostname或者内网ip地址（内网访问时使用）
host.name=192.168.142.157

# Hostname the broker will advertise to producers and consumers. If not set, it uses the
# value for "host.name" if configured.  Otherwise, it will use the value returned from
# java.net.InetAddress.getCanonicalHostName().
# 服务监听地址，一般配置为本机的hostname或者外网ip地址（外网访问时使用）
advertised.host.name=192.168.142.157

# The port to publish to ZooKeeper for clients to use. If this is not set,
# it will publish the same port that the broker binds to.
# 服务监听端口（外网访问时使用）
# 【注】：最新版本0.10.x broker配置已弃用了advertised.host.name和advertised.port ，改用advertised.listeners。
#advertised.port=<port accessible by clients>

# The number of threads handling network requests
# 处理网络请求的线程数
num.network.threads=3
 
# The number of threads doing disk I/O
# 处理磁盘I/O的线程数
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
# socket消息发送缓冲区
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
# socket消息接收缓冲区
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
# socket请求的最大字节数。（需要注意OutOfMemoryError的隐患，不得超过实际内存大小）
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma seperated list of directories under which to store log files
# 消息日志文件的存放位置，可配置多个，使用逗号分隔即可
log.dirs=/kafka/kafka-logs

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
# topic的分区数，分区越多，对应的segment就越多（实际在broker中存储的分散度也是基于此参数生成的）
num.partitions=12

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
# 在启动时恢复日志和关闭时刷日志时每个数据目录的线程的数量，默认1（如果服务器使用了RAID阵列，那么建议增大该配置项的值）
num.recovery.threads.per.data.dir=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk. 
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to exceessive seeks. 
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
# flush数据的上限消息量，当达到该消息数量时，会强制进行一次日志flush，将数据flush到日志文件中
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
# flush数据的上限时间间隔（单位：毫秒），当达到该时间间隔，会强制进行一次日志flush，将数据flush到日志文件中
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion
# 日志保存时间上限 (单位：hours|minutes)，默认为7天（168小时）；一旦超过这个限制就会根据policy处理数据。
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log as long as the remaining
# segments don't drop below log.retention.bytes.
# 日志保存大小上限 (单位：bytes)；一旦超过这个限制就会根据policy处理数据。
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
# segment文件的大小上限，超出该大小则会产生一个新的日志segment文件（-1表示不限制）
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according 
# to the retention policies
# 日志文件的检查周期，主要检查是否达到了删除策略的限制条件触发删除策略操作（log.retention.hours或log.retention.bytes）
log.retention.check.interval.ms=300000

# By default the log cleaner is disabled and the log retention policy will default to just delete segments after their retention expires.
# If log.cleaner.enable=true is set the cleaner will be enabled and individual logs can then be marked for log compaction.
# 默认是false的，配置为true的时候，会开启日志压缩；false状态下会直接删除segment（针对retention条件达到的时候的处理方式）
log.cleaner.enable=false

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
# Zookeeper连接方式配置（一般默认是2181端口，视具体情况而定）
zookeeper.connect=192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181

# Timeout in ms for connecting to zookeeper
# 连接Zookeeper的超时时间（单位：毫秒）
zookeeper.connection.timeout.ms=6000
[root@63c3077b732d kafka-test3-1]# 
```

## 补充配置项
```
#---------- 以下配置项是默认配置文件里面没有出现的 --------------#

############################# System #############################

# 后台线程数（一般不需要改动）
background.threads = 4

# IO线程处理的请求等待队列上限
queued.max.requests = 500
 
# 是否允许删除topic（关闭此配置项则通过shell进行topic的delete操作时是不生效的）
delete.topic.enable=true

############################# Log #################################

# 日志flush的检查周期，检查是否需要将日志进行flush（单位：毫秒）；与Log Flush Policy配合使用
log.flush.scheduler.interval.ms = 3000

# 日志清理策略（选项：delete|compact）；与日志到达retention条件后的处理相关
log.cleanup.policy = delete

# segment的滚动周期，当达到下面的时间，就会强制新建一个segment
log.roll.hours = 24*7
 
# 压缩的日志保留的最长时间（默认单位：毫秒，可指定单位）
log.cleaner.delete.retention.ms = 1 day
 
# segment日志的索引文件大小限制（单位：byte）
log.index.size.max.bytes = 10 * 1024 * 1024

# 索引计算的一个缓冲区，一般不需设置
log.index.interval.bytes = 4096
 
############################# Topic ##############################

# 是否允许自动创建topic ，若是false，就只能通过命令创建topic
auto.create.topics.enable =true

# 一个topic ，默认分区的replication（副本）个数 ，配置的值不能大于集群中broker的个数
default.replication.factor =1

# 消息体的最大大小（单位：byte）
message.max.bytes = 1000000

############################# Replica #############################

# partition管理控制器进行备份时，socket的超时时间
controller.socket.timeout.ms = 30000

# controller-to-broker-channels的buffer尺寸大小
controller.message.queue.size=10

# replicas响应leader的最长等待时间，若是超过这个时间，就将replicas排除在管理之外
replica.lag.time.max.ms = 10000

# 是否允许控制器关闭broker ,若是设置为true，会关闭所有在这个broker上的leader，并转移到其他broker
controlled.shutdown.enable = false

# 控制器关闭的最大重试次数
controlled.shutdown.max.retries = 3

# 每次关闭尝试的时间间隔
controlled.shutdown.retry.backoff.ms = 5000
 
# replicas消息的延迟时间上限，如果relicas落后太多,将会认为此partition relicas已经失效
# 一般情况下,因为网络延迟等原因,总会导致replicas中消息同步滞后
# 如果消息严重滞后,leader将认为此relicas网络延迟较大或者消息吞吐能力有限
# 在broker数量较少,或者网络不足的环境中,建议提高此值
replica.lag.max.messages = 4000

# leader与relicas的socket超时时间
replica.socket.timeout.ms= 30 * 1000

# leader复制的socket缓存大小
replica.socket.receive.buffer.bytes=64 * 1024

# replicas每次获取数据的最大字节数
replica.fetch.max.bytes = 1024 * 1024

# replicas同leader之间通信的最大等待时间，失败了会重试
replica.fetch.wait.max.ms = 500

# 每一个fetch操作的最小数据尺寸,如果leader中尚未同步的数据不足此值,将会等待直到数据达到这个大小
replica.fetch.min.bytes =1

# leader中进行复制的线程数，增大这个数值会增加relipca的IO
num.replica.fetchers = 1

# 每个replica将最高水位进行flush的时间间隔
replica.high.watermark.checkpoint.interval.ms = 5000
 
# 是否自动平衡broker之间的分配策略
auto.leader.rebalance.enable = false

# leader的不平衡比例，若是超过这个数值，会对分区进行重新的平衡
leader.imbalance.per.broker.percentage = 10

# 检查leader是否不平衡的时间间隔
leader.imbalance.check.interval.seconds = 300

# 客户端保留offset信息的最大空间大小
offset.metadata.max.bytes = 1024
 
#############################Consumer #############################
# Consumer端核心的配置是group.id、zookeeper.connect

# 决定该Consumer归属的唯一组ID
group.id

# 消费者的ID，若是没有设置的话，会自增
consumer.id

# 一个用于跟踪调查的ID ，最好同group.id相同
client.id = <group_id>
 
# zookeeper的心跳超时时间，查过这个时间就认为是无效的消费者
zookeeper.session.timeout.ms = 6000

# zookeeper的等待连接时间
zookeeper.connection.timeout.ms = 6000

# zookeeper的follower同leader的同步时间
zookeeper.sync.time.ms = 2000

# 当zookeeper中没有初始的offset时，或者超出offset上限时的处理方式 。
# smallest ：重置为最小值 
# largest:重置为最大值 
# anything else：抛出异常给consumer
auto.offset.reset = largest
 
# socket的超时时间，实际的超时时间为max.fetch.wait + socket.timeout.ms.
socket.timeout.ms= 30 * 1000

# socket的接收缓存空间大小
socket.receive.buffer.bytes=64 * 1024

#从每个分区fetch的消息大小限制
fetch.message.max.bytes = 1024 * 1024
 
# true时，Consumer会在消费消息后将offset同步到zookeeper，当Consumer失败后，新的consumer就能从zookeeper获取最新的offset
auto.commit.enable = true

# 自动提交的时间间隔
auto.commit.interval.ms = 60 * 1000
 
# 用于消费的最大数量的消息块缓冲大小，每个块可以等同于fetch.message.max.bytes中数值
queued.max.message.chunks = 10
 
# 当有新的consumer加入到group时,将尝试reblance,将partitions的消费端迁移到新的consumer中, 该设置是尝试的次数
rebalance.max.retries = 4

# 每次reblance的时间间隔
rebalance.backoff.ms = 2000

# 每次重新选举leader的时间
refresh.leader.backoff.ms
 
# server发送到消费端的最小数据，若是不满足这个数值则会等待直到满足指定大小。默认为1表示立即接收。
fetch.min.bytes = 1

# 若是不满足fetch.min.bytes时，等待消费端请求的最长等待时间
fetch.wait.max.ms = 100

# 如果指定时间内没有新消息可用于消费，就抛出异常，默认-1表示不受限
consumer.timeout.ms = -1
 
#############################Producer#############################
# 核心的配置包括：
# metadata.broker.list
# request.required.acks
# producer.type
# serializer.class
 
# 消费者获取消息元信息(topics, partitions and replicas)的地址,配置格式是：host1:port1,host2:port2，也可以在外面设置
metadata.broker.list
 
#消息的确认模式
# 0：不保证消息的到达确认，只管发送，低延迟但是会出现消息的丢失，在某个server失败的情况下，有点像TCP
# 1：发送消息，并会等待leader 收到确认后，一定的可靠性
# -1：发送消息，等待leader收到确认，并进行复制操作后，才返回，最高的可靠性
request.required.acks = 0
 
# 消息发送的最长等待时间
request.timeout.ms = 10000

# socket的缓存大小
send.buffer.bytes=100*1024

# key的序列化方式，若是没有设置，同serializer.class
key.serializer.class

# 分区的策略，默认是取模
partitioner.class=kafka.producer.DefaultPartitioner

# 消息的压缩模式，默认是none，可以有gzip和snappy
compression.codec = none

# 可以针对默写特定的topic进行压缩
compressed.topics=null

# 消息发送失败后的重试次数
message.send.max.retries = 3

# 每次失败后的间隔时间
retry.backoff.ms = 100

# 生产者定时更新topic元信息的时间间隔 ，若是设置为0，那么会在每个消息发送后都去更新数据
topic.metadata.refresh.interval.ms = 600 * 1000

# 用户随意指定，但是不能重复，主要用于跟踪记录消息
client.id=""
 
# 异步模式下缓冲数据的最大时间。例如设置为100则会集合100ms内的消息后发送，这样会提高吞吐量，但是会增加消息发送的延时
queue.buffering.max.ms = 5000

# 异步模式下缓冲的最大消息数，同上
queue.buffering.max.messages = 10000

# 异步模式下，消息进入队列的等待时间。若是设置为0，则消息不等待，如果进入不了队列，则直接被抛弃
queue.enqueue.timeout.ms = -1

# 异步模式下，每次发送的消息数，当queue.buffering.max.messages或queue.buffering.max.ms满足任一时producer会触发发送
batch.num.messages=200
```