---
title: Kafka实战系列（一）：Windows测试环境搭建
categories:
  - 消息和缓存系列之Kafka
tags:
  - Kafka
  - Big Data
comments: true
abbrlink: 17f0c072
date: 2018-08-23 16:20:51
---
【引言】所有工具的熟悉的第一步都是从安装环境开始的，所以kafka也不例外，虽然前面在Linux服务器上已经搭建过集群了，但自己玩的话，还是在本机搭个小环境比较方便。
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/public/000020.jpg" width="500">
<!-- more -->

# 安装前准备

## 安装zookeeper

### 下载zookeeper
&emsp;&emsp;官方版本发布下载地址：https://zookeeper.apache.org/releases.html ；其实是有跳转到Apache的，所以也可以直接到Apache官网这个页面下载： https://www.apache.org/dyn/closer.cgi/zookeeper/ ；选择好版本下载保存到本地。

### 安装zookeeper
- 将压缩包解压到本地磁盘（D:\Kafka）下；此时目录为：D:\Kafka\zookeeper-3.5.4-beta
- 进入D:\Kafka\zookeeper-3.5.4-beta\conf
- 将zoo_sample.cfg重命名为zoo.cfg（建议先备份一份）
- 用编辑器打开zoo.cfg，修改部分配置（主要是dataDir，当然如果你觉得需要也可以改改端口号）
```
...
# the directory where the snapshot is stored. do not use /tmp for storage, /tmp here is just example sakes.
dataDir=D:/Kafka/zookeeper-3.5.4-beta/data/zookeeper

# the port at which the clients will connect
clientPort=2181
...
```
- 添加系统环境变量（为了便于我们直接在cmd启动服务的，不配其实也不要紧）
```
ZOOKEEPER_HOME: D:\Kafka\zookeeper-3.5.4-beta
Path: 在最后后面添加;%ZOOKEEPER_HOME%\bin;（win10的话直接新建一个项就行了）
```

### 安装结果验证
&emsp;&emsp;在cmd启动zkServer，出现如下日志就表示启动成功了。
```
Microsoft Windows [版本 10.0.17134.165]
(c) 2018 Microsoft Corporation。保留所有权利。

C:\Users\chenglin>zkServer

C:\Users\chenglin>call "C:\Program Files\Java\jdk1.8.0_131"\bin\java " ......
......
2018-08-24 17:06:43,704 [myid:] - INFO  [main:QuorumPeerConfig@130] - Reading configuration from: D:\Kafka\zookeeper-3.5.4-beta\bin\..\conf\zoo.cfg
2018-08-24 17:06:43,705 [myid:] - INFO  [main:QuorumPeerConfig@376] - clientPortAddress is 0.0.0.0/0.0.0.0:2181
2018-08-24 17:06:43,705 [myid:] - INFO  [main:QuorumPeerConfig@380] - secureClientPort is not set
2018-08-24 17:06:43,705 [myid:] - INFO  [main:ZooKeeperServerMain@117] - Starting server
......
2018-08-24 17:06:53,616 [myid:] - INFO  [main:JettyAdminServer@112] - Started AdminServer on address 0.0.0.0, port 8080 and command URL /commands
......
2018-08-24 17:06:53,649 [myid:] - INFO  [main:NIOServerCnxnFactory@686] - binding to port 0.0.0.0/0.0.0.0:2181
......
2018-08-24 17:06:53,698 [myid:] - INFO  [main:ContainerManager@64] - Using checkIntervalMs=60000 maxPerMinute=10000
```

## 安装Kafka

### 下载kafka
&emsp;&emsp;官方版本发布下载地址： http://kafka.apache.org/downloads ；其实是有跳转到Apache的，所以也可以直接到Apache官网这个页面下载： https://www.apache.org/dyn/closer.cgi?path=/kafka/2.0.0/kafka_2.12-2.0.0.tgz ；选择好版本下载保存到本地。

### 安装Kafka
- 将压缩包解压到本地磁盘（D:\Kafka）下；此时目录为：D:\Kafka\kafka_2.12-2.0.0
- 进入D:\Kafka\kafka_2.12-2.0.0\config
- 用编辑器打开server.properties，修改部分配置（建议先备份一份；主要是log.dirs，当然如果你觉得需要也可以改改端口号什么的）
```
...
# 关于配置项的说明可以参照前面的一篇针对kafka的配置说明的专题文章
# A comma separated list of directories under which to store log files
log.dirs=D:/Kafka/zookeeper-3.5.4-beta/data/kafka
...
```

### 安装结果验证
&emsp;&emsp;打开cmd，进入D:\Kafka\kafka_2.12-2.0.0，并执行启动命令（必须先保证zookeeper是正常运行的）；出现如下日志就表示启动成功了。
```
Microsoft Windows [版本 10.0.17134.165]
(c) 2018 Microsoft Corporation。保留所有权利。

C:\Users\chenglin>D:

D:\>cd D:\Kafka\kafka_2.12-2.0.0

D:\Kafka\kafka_2.12-2.0.0>.\bin\windows\kafka-server-start.bat .\config\server.properties
...... 一大堆启动日志（zookeeper信息等等）
[2018-08-24 17:18:35,484] INFO [SocketServer brokerId=0] Started processors for 1 acceptors (kafka.network.SocketServer)
[2018-08-24 17:18:35,492] INFO Kafka version : 2.0.0 (org.apache.kafka.common.utils.AppInfoParser)
[2018-08-24 17:18:35,497] INFO Kafka commitId : 3402a8361b734732 (org.apache.kafka.common.utils.AppInfoParser)
[2018-08-24 17:18:35,516] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```

## Java开发环境
&emsp;&emsp;作为一个Java开发攻城狮，这是最基础最基础的环境了，就用不着多费口舌了。

# 命令模式测试

## Producer
```
Microsoft Windows [版本 10.0.17134.165]
(c) 2018 Microsoft Corporation。保留所有权利。

C:\Users\chenglin>cd D:\Kafka\kafka_2.12-2.0.0\bin\windows

C:\Users\chenglin>D:

D:\Kafka\kafka_2.12-2.0.0\bin\windows>kafka-console-producer.bat --broker-list localhost:9092 --topic demo
>333
>444
>555
>666
>
```

## Consumer
```
Microsoft Windows [版本 10.0.17134.165]
(c) 2018 Microsoft Corporation。保留所有权利。

C:\Users\chenglin>cd D:\Kafka\kafka_2.12-2.0.0\bin\windows

C:\Users\chenglin>D:

D:\Kafka\kafka_2.12-2.0.0\bin\windows>kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic demo
333
444
555
666
```

## 特殊说明
&emsp;&emsp;网上很多资料都是以--zookeeper参数的方式来启动Consumer，实际上，在新版本的kafka（据说是0.90以后）已经废弃了这个--zookeeper参数了，所以这么启动会报错的（zookeeper is not a recognized option）；替代方法如下：
```
已废弃：kafka-console-consumer.bat --zookeeper localhost:2181 --topic demo
替代方法：kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic demo
```

# 小总结
&emsp;&emsp;现在类似于zookeeper、kafka这种软件基本都做到了开箱即用，配置集群也是很方便的；这里搭建这个环境主要是为了后面的Demo验证使用的，所以不追求那么完美。实际应用时肯定有很多参数调整，目录规划，日志级别和分类调整之类的操作的。