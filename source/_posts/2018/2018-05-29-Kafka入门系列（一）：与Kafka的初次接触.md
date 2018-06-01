---
title: Kafka入门系列（一）：与Kafka的初次接触
comments: true
categories:
  - Big Data Component - Kafka
tags:
  - Kafka
  - Big Data
abbrlink: 7abadfc4
date: 2018-05-29 19:09:28
---
【引言】本篇为Kafka入门系列的第一讲，主要讲解Kafka的基本概念和基础结构，让初学者有个初步的认识。
<div align=center><img src="/img/2018-05-29-01.png" width="500"/></div>
<!-- more -->

# 基本概念

## 官方描述
Apache Kafka® is a distributed streaming platform. What exactly does that mean?A streaming platform has three key capabilities:
+ Publish and subscribe to streams of records, similar to a message queue or enterprise messaging system.
+ Store streams of records in a fault-tolerant durable way.
+ Process streams of records as they occur.
------
Kafka is generally used for two broad classes of applications:
+ Building real-time streaming data pipelines that reliably get data between systems or applications
+ Building real-time streaming applications that transform or react to the streams of data
------
First a few concepts:
+ Kafka is run as a cluster on one or more servers that can span multiple datacenters.
+ The Kafka cluster stores streams of records in categories called topics.
+ Each record consists of a key, a value, and a timestamp.

## 通俗理解
+ Kafka是一个开源的、基于发布-订阅模式的、可热扩展的、分布式的消息系统（Message Queue）
+ Kafka可同时支持离线数据处理和实时数据处理（比如：用于hadoop或者storm）
+ Kafka可以时间复杂度为O(1)的方式提供消息持久化能力，并保证低配服务器和大数据量的情况下的访问性能稳定
+ Kafka有很好的容错性，允许集群中节点失败（若副本数量为n,则允许n-1个节点失败）
+ Kafka对高并发量的请求有很好的支持性，可轻松支持数千个客户端的同时读写

## Core APIs
<img style="clear: both;display: block;margin:auto;" src="/img/2018-06-01-05.png" width="50%">
+ The Producer API allows an application to publish a stream of records to one or more Kafka topics.（生产者API）
+ The Consumer API allows an application to subscribe to one or more topics and process the stream of records produced to them.（消费者API）
+ The Streams API allows an application to act as a stream processor, consuming an input stream from one or more topics and producing an output stream to one or more output topics, effectively transforming the input streams to output streams.（数据流处理API）
+ The Connector API allows building and running reusable producers or consumers that connect Kafka topics to existing applications or data systems. For example, a connector to a relational database might capture every change to a table.（数据连接连接API）

# 架构分析

## 数据流向总览
<img style="clear: both;display: block;margin:auto;" src="/img/2018-06-01-02.png" width="75%">
+ Producers：消息的生产者
+ Brokers：kafka集群（消息的传输和存储者）
+ Consumers：消息的消费者

## 数据流向细节
<img style="clear: both;display: block;margin:auto;" src="/img/2018-06-01-01.jpg" width="75%">
+ Topic：消息的主题
+ Consumer Group：消费者分组
+ Partition：消息的分片

# 核心概念

## Broker
&emsp;&emsp;Kafka集群中一般包含多个服务器（也可以是一个），这类服务器被称为broker；broker的原意是中间人、代理人，实际在Kafka集群中，也可以类似的理解，broker作为消息传递的中间人，负责了producer和consumer直接的消息传递（类似于一个快递中心的角色）

## Producer
&emsp;&emsp;从字面理解，也很通俗易懂，就是消息的生产者；producer定义好消息的格式，消息的主题（topic），消息的内容，然后发送给了Kafka集群（实际就是broker来完成消息的接收和持久化），至此producer的工作就结束了，至于后面消息怎么存，什么人回来消费，那都不是producer该关心的了

## Consumer
&emsp;&emsp;和前一个概念一样，这也是个很容易从字面意思就能明白用途的角色，它就是消息的消费者；通常来讲，消费才是促进生产的原动力，在kafka里面也一样，只有有了消费的需求，才有必要生产，否则生产出来的消息没有人来消费就都成为垃圾数据了

## Kafka Cluster
&emsp;&emsp;Kafka既然是分布式的消息系统，那么必须是要支持以集群的模式存在的，通常我们会对架构的描述有三个常见名词：单机、集群、分布式；这三者的概念大概可以按下面的特征划分（图片来源于知乎）
+ 单机：所有业务都在一台服务器上，相对于集群模式（从物理结构上可区分）
+ 集群：相同的一个业务，部署在多个服务器上，相对于单机模式（从物理结构上可区分）
+ 分布式：一个业务分拆多个子业务，相对于集群来说集群描述的是物理形态，分布式描述的是一种工作方式。
<img style="clear: both;display: block;margin:auto;" src="/img/2018-06-01-03.png" width="50%">

## Topic
&emsp;&emsp;topic的字面意思就是话题；在kafka的世界里面，可以理解为producer和consumer之间聊天的日常，两个人聊天就肯定会涉及到一个相同的话题，这个话题就是所谓的topic，也就是说只有关于这个topic的消息，才能在围绕这个topic的producer和consumer之间传递（其他消息consumer根本不用关心）；实际情况可能比这个比喻更为复杂（因为实际上一个话题可以被若干的consumer消费，一个producer也可以在不同话题之间切换）

## Partition
&emsp;&emsp;parition的字面意义是分割，在kafka体系里对应的是物理上的分割的概念，创建topic的时候可以指定该topic包含一个或多个partition，而在实际的存储中每个partition对应的是一个文件夹，这个文件夹下存储的就是该partition被分配到的的数据和索引文件，每个partition都是一个有序的队列（有自己的offset）

## Consumer Group
&emsp;&emsp;消费者分组，这是kafka用来实现一个topic的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段。一个topic可以有多个Consumer Group。topic里的消息会给所有的Consumer Group都分发一份（可以理解为复制，但不是真的复制），但每个partion只会把消息发给该Consumer Group中的某一个consumer。
&emsp;&emsp;如果需要实现广播，只要每个consumer有一个独立的Consumer Group就可以了（这样每个consumer都可以接收全量的消息）。要实现单播只要所有的consumer在同一个Consumer Group（这样就是所有的consumer来share全量的消息）。

## Zookeeper
&emsp;&emsp;ZooKeeper本身是一种分布式协调服务，用于管理大型主机。在分布式环境中协调和管理服务是一个复杂的过程。ZooKeeper通过其简单的架构和API解决了这个问题。ZooKeeper允许开发人员专注于核心应用程序逻辑，而不必担心应用程序的分布式特性。作为去中心化的集群模式；需要要消费者知道现在那些生产者（对于消费者而言，kafka就是生产者）是可用的，这也就是Zookeeper的存在价值，它是用来做分布式集群管理的。
&emsp;&emsp;实际应用中，Kafka将元数据信息保存在Zookeeper中，但是发送给Topic本身的数据是不会发到Zookeeper上的，否则Zookeeper就会爆掉了。kafka使用zookeeper可以实现动态的集群扩展，而不需要更改客户端（producer和consumer）的任何配置（对外来说，集群是一个整体，集群内部的扩展对外是不可感知的）。broker会在zookeeper注册并保持相关的元数据（topic，partition信息等）更新。Zookeeper中kafka节点的元数据存储结构如下：
<img style="clear: both;display: block;margin:auto;" src="/img/2018-06-01-04.png" width="90%">
