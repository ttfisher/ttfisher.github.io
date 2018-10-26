---
title: Kafka入门系列（四）：Kafka配置文件详解
categories:
  - 【104】风住尘香花已尽之消息队列和缓存
tags:
  - Kafka
  - Big Data
comments: true
abbrlink: d67d13cd
date: 2018-05-31 19:09:28
---
【引言】本篇为Kafka入门系列的第四讲，从配置中深入一点去理解Kafka！
<div align=center><img src="/img/2018/2018-05-31-01.jpg" width="500"/></div>
<!-- more -->

# 默认配置项（kafka_2.10-0.9.0.0）
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

# 补充配置项
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