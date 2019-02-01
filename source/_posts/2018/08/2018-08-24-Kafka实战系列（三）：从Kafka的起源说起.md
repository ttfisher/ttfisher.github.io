---
title: Kafka实战系列（三）：从Kafka的起源说起（未完成...)
comments: true
categories:
  - 消息和缓存系列之Kafka
tags:
  - Kafka
  - Big Data
abbrlink: ad8a452
date: 2018-08-24 16:25:55
---
【引言】同样是作为消息中间件（虽然具体的应用场景不同），为什么Kafka优于其他的中间件呢？都说Kafka高性能，具体为什么会有那么高的性能呢？除了在软件本身层面的设计优化，其他层面是否有新的技术运用呢？针对这一系列的问题，就通过这个进阶系列的专题来详细的探究探究......
<img style="clear: both;display: block;margin:auto;" src="http://pm4hdun71.bkt.clouddn.com/img/2018/2018-08-24-02.jpg" width="500">
<!-- more -->

# Kafka的特性

## 特性列举
&emsp;&emsp;在研究Kafka为什么比其他MQ快之前呢，我们先来看看Kafka有哪些特性（其中可能有部分概念目前还不熟悉，后面会逐一补充解释）：
+ 通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能。
+ 高吞吐量：即使是非常普通的硬件kafka也可以支持每秒数十万的消息。
+ 支持同步和异步复制两种HA
+ Consumer客户端pull，随机读,利用sendfile系统调用，zero-copy ,批量拉数据
+ 消费状态保存在客户端
+ 消息存储顺序写
+ 数据迁移、扩容对用户透明
+ 支持Hadoop并行数据加载。
+ 支持online和offline的场景。
+ 持久化：通过将数据持久化到硬盘以及replication防止数据丢失。
+ scale out：无需停机即可扩展机器。
+ 定期删除机制，支持设定partitions的segment file保留时间。

## 重点说明

### Kafka如何保证消息的可靠送达
&emsp;&emsp;传统的MQ系统通常都是通过broker和consumer间的确认（ack）机制实现的，并在broker保存消息分发的状态。即使这样一致性也是很难保证的。而kafka的做法是由consumer自己保存状态，也不要做任何的确认。这样虽然consumer负担更重，但其实更灵活了。因为不管consumer上任何原因导致需要重新处理消息，都可以再次从broker获得。

### Kafka的系统扩展性如何保证
&emsp;&emsp;kafka使用zookeeper来实现动态的集群扩展（现在很多大数据组件都是用这一套），所以kafka集群的变动对外是无感知的，也不需要更改客户端（producer和consumer）的配置；broker会在zookeeper注册并保持相关的元数据（topic，partition信息等）更新。而客户端会在zookeeper上注册相关的watcher。一旦zookeeper发生变化，客户端能及时感知并作出相应调整。这样就保证了添加或去除broker时，各broker间仍能自动实现负载均衡。
> ACK: Acknowledgement的缩写，即确认字符，在数据通信中，接收站发给发送站的一种传输类控制字符。表示发来的数据已确认接收无误。

### Kafka的核心设计目标
+ 数据磁盘持久化：消息不在内存中cache，直接写入到磁盘，充分利用磁盘的顺序读写性能。
+ zero-copy：减少IO操作步骤。
+ 支持数据批量发送和拉取。
+ 支持数据压缩。
+ Topic划分为多个partition，提高并行处理能力。

### Producer相关特性
+ producer会根据用户指定的算法，将消息发送到指定的partition
+ 一般都会有多个partition，而每个partition都有自己的replica（分布在不同的broker上）
+ 多个partition会由系统选举出一个leader，由leader来负责读写，但是状态管理（fail over）由zookeeper来控制
+ 所有相关节点（比如：broker）加入和退出都由zookeeper来管理

### Consumer的相关特性
+ 由于Kafka不是缓存型的，它的数据是持久化的，所以consumer采用pull的方式消费数据
+ consumer本身可根据自身能力控制消费的频度
+ consumer可自由选择消费模式：批量消费、从固定位置消费（offset）

