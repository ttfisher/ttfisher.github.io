---
title: Kafka入门系列（二）：Kafka集群的安装和配置
comments: true
categories:
  - 消息和缓存系列之Kafka
tags:
  - Kafka
  - Big Data
abbrlink: f98289eb
date: 2018-05-30 16:55:45
---
【引言】本篇为Kafka入门系列的第二讲，经过第一讲针对Kafka的基础概念和基本结构讲解，我们已经对Kafka有了一个粗略的认识，本篇即开始在实践中来熟悉和理解Kafka，实践的第一步就是安装和配置的过程。
<div align=center><img src="http://pm4hdun71.bkt.clouddn.com/img/2018/2018-05-30-01.jpg" width="500"/></div>
<!-- more -->

# 安装说明
&emsp;&emsp;Kafka集群需要安装的基础组件如下：
+ JDK：基础Java环境，Zookeeper和Kafka对Java基础环境都有一定的依赖，所以JDK是必备的
+ Zookeeper：Zookeeper在大数据处理上经常用到，主要是用于Kafka的状态保存；后面也会起一个专栏专门研究一下Zookeeper。（官方解释ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services.）
+ Kafka：Kafka和很多大数据组件一样都是解压即用的（当然该修改的配置文件还是要改的）

# 准备工作

## 服务器准备
&emsp;&emsp;这次我们准备搭建一个3节点的小集群，鉴于创建虚拟机比较费时一些，所以这次我使用的是docker来搭建的。具体的机器分配情况如下表所示：

|docker名|操作系统|hostname|IP地址|备注|
|  :--:  |  :--:  |  :--:  | :--: |:--:|
|cl_76_153|centos7.4|1bc72703f1e9|192.168.142.153|Kafka01、Zookeeper01|
|cl_76_156|centos7.4|b3e6614d0345|192.168.142.156|Kafka02、Zookeeper02|
|cl_76_157|centos7.4|63c3077b732d|192.168.142.157|Kafka03、Zookeeper03|
【问】：为什么hostname看上去那么诡异呢？
【答】：因为这个就是docker生成后的一个随机ID，尝试过在docker里面修改hostname，但是重启之后就又被恢复回去了，后面会单独开个研究docker的专题，再来解决这个疑问。

## 组件包准备
|组件名称|发行版本|备注|
|  :--:  |  :--:  |:--:|
|JDK|1.7.0_79|因验证2016年的项目系统，没有用最新版|
|Zookeeper|3.4.6|因验证2016年的项目系统，没有用最新版|
|Kafka|2.10-0.9.0.0|因验证2016年的项目系统，没有用最新版|

## 安装流程
1. 下载安装源：JDK、Zookeeper、Kafka
2. 分发安装源到各个集群节点
3. 分别按顺序完成安装：JDK - Zookeeper - Kafka
4. 完成环境变量配置（主要针对JDK）
5. 完成Zookeeper和Kafka配置
6. 按步骤启动服务（Zookeeper - Kafka）
7. 完成基本流程验证

# 安装JDK

## 手工安装
+ 将jdk-7u79-linux-x64.tar.gz上传到/opt/tools/目录下
+ 切换到/opt/tools目录下，解压安装源，解压完成后删除安装源：
```
[root@centos154 ~]#cd /opt/tools
[root@centos154 ~]#tar –zxvf ./jdk-7u79-linux-x64.tar.gz
[root@centos154 ~]#rm ./jdk-7u79-linux-x64.tar.gz
```
+ 修改环境变量配置(追加)，并立即重加载环境变量：
```
[root@centos154 ~]#vi /etc/profile
export JAVA_HOME=/opt/tools/jdk1.7.0_79
export CLASSPATH=${JAVA_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
...
[root@centos154 ~]# source /etc/profile
```
+ 测试安装是否成功：
```
[root@centos154 ~]# java -version
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
[root@centos154 ~]# 
```

## 自动安装
&emsp;&emsp;可以通过任意工具将JDK安装包（jdk-7u79-linux-x64.tar.gz）上传到服务器上，然后将如下脚本存档为一个sh文件并上传到安装包的同级目录，最后为脚本添加权限后执行即可完成安装（必须保证执行脚本的用户为root权限用户，因为有对系统配置文件的操作，若是ubuntu系统可考虑使用-i切换到root）。脚本仅供参考，可视具体使用场景调整。
```
#!/bin/bash

#脚本所在目录
script_dir=$( cd "$( dirname "$0" )" && pwd )

#若未安装jdk，安装jdk
JAVA=$(which java)
if [[ -z "$JAVA" ]]; then
    echo "【INFO】开始安装jdk..."
    #删除现有的jdk
    rm -rf /opt/tools/jdk1.7.0_79
    #解压安装jdk
    tar -zxvf ./jdk-7u79-linux-x64.tar.gz -C /opt/tools/
    #配置环境变量    
    ###  -------------- 删除原配置 --------------------  ###
    sed -i -e '/JAVA_HOME/d' -e '/JRE/d' /etc/profile
    ###  -------------- 添加新配置 --------------------  ###
    sed -i '$a## ---------------------  JAVA_HOME -------------------------- ##' /etc/profile
    sed -i '$aJAVA_HOME=/opt/tools/jdk1.7.0_79' /etc/profile
    sed -i '$aJRE_HOME=/opt/tools/jdk1.7.0_79/jre' /etc/profile
    sed -i '$aPATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin' /etc/profile
    sed -i '$aCLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib' /etc/profile
    sed -i '$aexport JAVA_HOME JRE_HOME PATH CLASSPATH' /etc/profile
    echo "$JAVA_HOME"
    ### --------------- 使环境变量生效 ----------------  ###
    source /etc/profile
    sleep 1
    echo "【SUCCESS】：安装JDK完成"
fi

echo "**************************************************"
echo "【java -version】："
java -version
```
【注】：通常在自动安装完成后需要手工执行一下如下命令：source /etc/profile；或者重新登录服务器。

# 安装Zookeeper

## 安装步骤

+ 配置host映射
```
[root@1bc72703f1e9 ~]# cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.6      1bc72703f1e9

# 此处的映射是为了在配置zookeeper和kafka时可以直接使用集群各主机的hostname来进行通信
# 配置完成后需要ping一下是否可通信成功
192.168.142.153 1bc72703f1e9
192.168.142.156 b3e6614d0345
192.168.142.157 63c3077b732d

[root@1bc72703f1e9 ~]# ping b3e6614d0345
PING b3e6614d0345 (192.168.142.156) 56(84) bytes of data.
64 bytes from b3e6614d0345 (192.168.142.156): icmp_seq=1 ttl=64 time=0.136 ms
64 bytes from b3e6614d0345 (192.168.142.156): icmp_seq=2 ttl=64 time=0.072 ms
...
```

+ 创建数据目录，上传安装包并解压
```
[root@1bc72703f1e9 ~]# mkdir -p /zookeeper/data
[root@1bc72703f1e9 ~]# mv /opt/tools/zookeeper-3.4.6.tar.gz  /zookeeper/
[root@1bc72703f1e9 ~]# cd /zookeeper/
[root@1bc72703f1e9 zookeeper]# tar -zxvf zookeeper-3.4.6.tar.gz 
```

+ 创建配置文件（从安装包模板复制）
```
[root@1bc72703f1e9 zookeeper]# cd /zookeeper/zookeeper-3.4.6/conf
[root@1bc72703f1e9 conf]# cp zoo_sample.cfg zoo.cfg
[root@1bc72703f1e9 conf]# vi zoo.cfg 
...
```

+ 修改配置文件zoo.cfg
```
# 原有配置，修改即可
dataDir=/zookeeper/data

# 若原配置文件不存在，则新增此段（类似1bc72703f1e9的字串是集群的主机hostname）
server.1=1bc72703f1e9:2777:3777
server.2=b3e6614d0345:2777:3777
server.3=63c3077b732d:2777:3777
```

+ 分别在集群的各台主机完成安装和配置
```
# 建议直接scp已配置好的一台主机的Zookeeper目录，然后只需要修改一下myid即可
[root@1bc72703f1e9 /]# scp -r zookeeper/ root@192.168.142.156:/
root@192.168.142.157's password: 
...
[root@1bc72703f1e9 /]# scp -r zookeeper/ root@192.168.142.156:/
root@192.168.142.157's password: 
...
```

+ 分别设置各主机的myid
&emsp;&emsp;这个myid的值从哪里得来的呢？注意一下前面的zoo.cfg里面的这个配置：server.1=1bc72703f1e9:2777:3777，其中本主机对应的server后面的那个数字就是本机ID
```
# 主机1
[root@1bc72703f1e9 /]# echo "1">/zookeeper/data/myid

# 主机2
[root@b3e6614d0345 /]# echo "2">/zookeeper/data/myid

# 主机3
[root@63c3077b732d /]# echo "3">/zookeeper/data/myid
```

+ 分别启动3台主机的Zookeeper服务
```
# 启动服务
[root@63c3077b732d /]# cd /zookeeper/zookeeper-3.4.6/bin/
[root@63c3077b732d bin]# ./zkServer.sh stop
JMX enabled by default
Using config: /zookeeper/zookeeper-3.4.6/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
[root@63c3077b732d bin]# ./zkServer.sh start
JMX enabled by default
Using config: /zookeeper/zookeeper-3.4.6/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

# 查看状态 -- 3台主机的Mode应该是一个leader两个follower，至此即表示启动成功
[root@63c3077b732d bin]# ./zkServer.sh status
JMX enabled by default
Using config: /zookeeper/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: follower
[root@63c3077b732d bin]# 
```

## 特别提醒
+ **<font color=red>每台主机的/etc/hosts文件都必须要修改，否则服务启动会报如下异常：</font>**
```
2018-05-31 06:28:51,175 [myid:3] - WARN  [QuorumPeer[myid=3]/0.0.0.0:2181:QuorumCnxManager@382] 
- Cannot open channel to 1 at election address 1bc72703f1e9:2778
java.net.UnknownHostException: 1bc72703f1e9
        at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:178)
        at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
        at java.net.Socket.connect(Socket.java:579)
        at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectOne(QuorumCnxManager.java:368)
        at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectAll(QuorumCnxManager.java:402)
        at org.apache.zookeeper.server.quorum.FastLeaderElection.lookForLeader(FastLeaderElection.java:840)
        at org.apache.zookeeper.server.quorum.QuorumPeer.run(QuorumPeer.java:762)
```
+ **<font color=red>Zookeeper error: Cannot open channel to X at election address的问题起因</font>**
```
# How have defined the ip of the local server in each node? 
# If you have given the public ip, then the listener would have failed to connect to the port. 
# You must specify 0.0.0.0 for the current node

server.1=0.0.0.0:2777:2778
server.2=b3e6614d0345:3777:3778
server.3=63c3077b732d:4777:4778
```
+ **<font color=red>别忘了，我们用的是Docker，所以从端口使用上来讲，必须要避免重复</font>**
+ **<font color=red>关于Zookeeper启动的日志，一般情况下在bin目录的zookeeper.out文件查看</font>**

# 安装Kafka

+ 上传和解压安装包
```
[root@1bc72703f1e9 bin]# mkdir /kafka
[root@1bc72703f1e9 bin]# cd /kafka/
[root@1bc72703f1e9 kafka]# cp /opt/tools/kafka_2.10-0.9.0.0.tgz ./
[root@1bc72703f1e9 kafka]# tar -zxvf kafka_2.10-0.9.0.0.tgz 
...
```
+ 修改配置文件server.properties
```
[root@1bc72703f1e9 config]# pwd
/kafka/kafka_2.10-0.9.0.0/config
[root@1bc72703f1e9 config]# vi server.properties 
...
############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=1

############################# Socket Server Settings #############################

listeners=PLAINTEXT://:9092

# The port the socket server listens on
port=9092

# Hostname the broker will bind to. If not set, the server will bind to all interfaces
host.name=192.168.142.153

# Hostname the broker will advertise to producers and consumers. If not set, it uses the
# value for "host.name" if configured.  Otherwise, it will use the value returned from
# java.net.InetAddress.getCanonicalHostName().
advertised.host.name=192.168.142.153

# The default number of log partitions per topic. More partitions allow greater
num.partitions=12

# The number of threads handling network requests -- 不建议改动（生产环境可放大，比如16）
num.network.threads=3

...

############################# Log Basics #############################

# A comma seperated list of directories under which to store log files
log.dirs=/kafka/kafka-logs

...
...

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=6000

# 原始配置文件中并没有，新增的
delete.topic.enable=true
...
```

+ 同步kafka安装目录到其他机器
```
# 建议直接scp已配置好的一台主机的Kafka目录，然后只需要修改一下server.properties即可
[root@1bc72703f1e9 /]# scp -r kafka/ root@192.168.142.156:/
root@192.168.142.157's password: 
...
[root@1bc72703f1e9 /]# scp -r kafka/ root@192.168.142.156:/
root@192.168.142.157's password: 
...
```

+ 修改各台服务器的配置文件，修改项：broker.id、host.name、advertised.host.name
+ 启动kafka服务
```
[root@1bc72703f1e9 config]# cd /kafka/kafka_2.10-0.9.0.0/bin
[root@1bc72703f1e9 bin]# nohup ./kafka-server-start.sh /kafka/kafka_2.10-0.9.0.0/config/server.properties & 
[1] 1459
[root@1bc72703f1e9 bin]# nohup: ignoring input and appending output to 'nohup.out'

[root@1bc72703f1e9 bin]# tail -200f nohup.out 
[2018-05-31 08:06:15,576] INFO KafkaConfig values: 
        request.timeout.ms = 30000
        log.roll.hours = 168
        inter.broker.protocol.version = 0.9.0.X
        log.preallocate = false
        security.inter.broker.protocol = PLAINTEXT
        controller.socket.timeout.ms = 30000
        ssl.keymanager.algorithm = SunX509
        ssl.key.password = null
        log.cleaner.enable = false
        ssl.provider = null
        num.recovery.threads.per.data.dir = 1
        background.threads = 10
        unclean.leader.election.enable = true
        sasl.kerberos.kinit.cmd = /usr/bin/kinit
        replica.lag.time.max.ms = 10000
        ssl.endpoint.identification.algorithm = null
        auto.create.topics.enable = true
        zookeeper.sync.time.ms = 2000
        ssl.client.auth = none
        ssl.keystore.password = null
        log.cleaner.io.buffer.load.factor = 0.9
        offsets.topic.compression.codec = 0
        log.retention.hours = 168
        log.dirs = /kafka/kafka-logs
        ssl.protocol = TLS
        log.index.size.max.bytes = 10485760
        sasl.kerberos.min.time.before.relogin = 60000
        log.retention.minutes = null
        connections.max.idle.ms = 600000
        ssl.trustmanager.algorithm = PKIX
        offsets.retention.minutes = 1440
        max.connections.per.ip = 2147483647
        replica.fetch.wait.max.ms = 500
        metrics.num.samples = 2
        port = 9092
        offsets.retention.check.interval.ms = 600000
        log.cleaner.dedupe.buffer.size = 524288000
        log.segment.bytes = 1073741824
        group.min.session.timeout.ms = 6000
        producer.purgatory.purge.interval.requests = 1000
        min.insync.replicas = 1
        ssl.truststore.password = null
        log.flush.scheduler.interval.ms = 9223372036854775807
        socket.receive.buffer.bytes = 102400
        leader.imbalance.per.broker.percentage = 10
        num.io.threads = 8
        zookeeper.connect = 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181
        queued.max.requests = 500
        offsets.topic.replication.factor = 3
        replica.socket.timeout.ms = 30000
        offsets.topic.segment.bytes = 104857600
        replica.high.watermark.checkpoint.interval.ms = 5000
        broker.id = 1
        ssl.keystore.location = null
        listeners = PLAINTEXT://:9092
        log.flush.interval.messages = 9223372036854775807
        principal.builder.class = class org.apache.kafka.common.security.auth.DefaultPrincipalBuilder
        log.retention.ms = null
        offsets.commit.required.acks = -1
        sasl.kerberos.principal.to.local.rules = [DEFAULT]
        group.max.session.timeout.ms = 30000
        num.replica.fetchers = 1
        advertised.listeners = null
        replica.socket.receive.buffer.bytes = 65536
        delete.topic.enable = true
        log.index.interval.bytes = 4096
        metric.reporters = []
        compression.type = producer
        log.cleanup.policy = delete
        controlled.shutdown.max.retries = 3
        log.cleaner.threads = 1
        quota.window.size.seconds = 1
        zookeeper.connection.timeout.ms = 6000
        offsets.load.buffer.size = 5242880
        zookeeper.session.timeout.ms = 6000
        ssl.cipher.suites = null
        authorizer.class.name = 
        sasl.kerberos.ticket.renew.jitter = 0.05
        sasl.kerberos.service.name = null
        controlled.shutdown.enable = true
        offsets.topic.num.partitions = 50
        quota.window.num = 11
        message.max.bytes = 1000012
        log.cleaner.backoff.ms = 15000
        log.roll.jitter.hours = 0
        log.retention.check.interval.ms = 300000
        replica.fetch.max.bytes = 1048576
        log.cleaner.delete.retention.ms = 86400000
        fetch.purgatory.purge.interval.requests = 1000
        log.cleaner.min.cleanable.ratio = 0.5
        offsets.commit.timeout.ms = 5000
        zookeeper.set.acl = false
        log.retention.bytes = -1
        offset.metadata.max.bytes = 4096
        leader.imbalance.check.interval.seconds = 300
        quota.consumer.default = 9223372036854775807
        log.roll.jitter.ms = null
        reserved.broker.max.id = 1000
        replica.fetch.backoff.ms = 1000
        advertised.host.name = 192.168.142.153
        quota.producer.default = 9223372036854775807
        log.cleaner.io.buffer.size = 524288
        controlled.shutdown.retry.backoff.ms = 5000
        log.dir = /tmp/kafka-logs
        log.flush.offset.checkpoint.interval.ms = 60000
        log.segment.delete.delay.ms = 60000
        num.partitions = 12
        num.network.threads = 3
        socket.request.max.bytes = 104857600
        sasl.kerberos.ticket.renew.window.factor = 0.8
        log.roll.ms = null
        ssl.enabled.protocols = [TLSv1.2, TLSv1.1, TLSv1]
        socket.send.buffer.bytes = 102400
        log.flush.interval.ms = null
        ssl.truststore.location = null
        log.cleaner.io.max.bytes.per.second = 1.7976931348623157E308
        default.replication.factor = 1
        metrics.sample.window.ms = 30000
        auto.leader.rebalance.enable = true
        host.name = 192.168.142.153
        ssl.truststore.type = JKS
        advertised.port = null
        max.connections.per.ip.overrides = 
        replica.fetch.min.bytes = 1
        ssl.keystore.type = JKS
 (kafka.server.KafkaConfig)
[2018-05-31 08:06:15,665] INFO starting (kafka.server.KafkaServer)
[2018-05-31 08:06:15,673] INFO Connecting to zookeeper on 
192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 (kafka.server.KafkaServer)
[2018-05-31 08:06:15,701] INFO Starting ZkClient event thread. (org.I0Itec.zkclient.ZkEventThread)
[2018-05-31 08:06:15,712] INFO Client environment:zookeeper.version=3.4.6-1569965, 
built on 02/20/2014 09:09 GMT (org.apache.zookeeper.ZooKeeper)
......
[2018-05-31 08:07:28,597] INFO Registered broker 2 at path /brokers/ids/2 with addresses: 
PLAINTEXT -> EndPoint(192.168.142.156,9092,PLAINTEXT) (kafka.utils.ZkUtils)
[2018-05-31 08:07:28,619] INFO Kafka version : 0.9.0.0 (org.apache.kafka.common.utils.AppInfoParser)
[2018-05-31 08:07:28,620] INFO Kafka commitId : fc7243c2af4b2b4a
(org.apache.kafka.common.utils.AppInfoParser)
[2018-05-31 08:07:28,621] INFO [Kafka Server 2], started (kafka.server.KafkaServer)
^C
[root@1bc72703f1e9 bin]# 
```

# 安装结果验证

## 创建topic
```
./kafka-topics.sh --create --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 --replication-factor 3 --partitions 16 --topic kafka-test
```

## 查看所有topic
```
./kafka-topics.sh --list --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181
```

## 创建producer
```
./kafka-console-producer.sh -broker-list 192.168.142.153:9092,192.168.142.156:9092,192.168.142.157:9092 -topic kafka-test
```

## 创建consumer
```
./kafka-console-consumer.sh --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 -topic kafka-test
```

## 消息收发流程验证

### producer消息发送过程
```
[root@63c3077b732d bin]# ./kafka-topics.sh --create --zookeeper  192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 --replication-factor 3 --partitions 16 --topic kafka-test
Created topic "kafka-test".
[root@63c3077b732d bin]# 
[root@63c3077b732d bin]# ./kafka-topics.sh --list --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 kafka-test
[root@63c3077b732d bin]# 
[root@63c3077b732d bin]# ./kafka-console-producer.sh -broker-list 192.168.142.153:9092,192.168.142.156:9092,192.168.142.157:9092 -topic kafka-test
Hello kafka
I am from Nanjing
```

### consumer消息接收过程
```
[root@b3e6614d0345 bin]# ./kafka-console-consumer.sh --zookeeper 192.168.142.153:2181,192.168.142.156:2181,192.168.142.157:2181 -topic kafka-test
Hello kafka
I am from Nanjing

```