---
title: Zookeeper菜鸟实录（一）：入门和安装
categories:
  - 分布式服务框架之Zookeeper
tags:
  - Zookeeper
comments: true
abbrlink: 7a8febe
date: 2018-11-05 11:48:43
---
【引言】已经记不得第一次接触Zookeeper是什么时候了，如今很多的分布式系统都会用到这个家伙，但实则对Zookeeper这个东西，如今是熟悉又陌生，感觉很多年前就认识了，但让我说说它的话，甚至都不知道从何说起；故而借着这次尝试了解solr的机会，先探究探究Zookeeper。
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2018/2018-11-12-01.jpg" width="55%"/></div>
<!-- more -->

# 概念

## What is ZooKeeper?
&emsp;&emsp;ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or another by distributed applications. Each time they are implemented there is a lot of work that goes into fixing the bugs and race conditions that are inevitable. Because of the difficulty of implementing these kinds of services, applications initially usually skimp on them ,which make them brittle in the presence of change and difficult to manage. Even when done correctly, different implementations of these services lead to management complexity when the applications are deployed.(From http://zookeeper.apache.org/ )

## 通俗的说
&emsp;&emsp;ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置管理，域名/命名服务，分布式同步以及集群/组管理。
&emsp;&emsp;虽然是以我们熟悉的母语重新对Zookeeper进行了一次描述，但是看上去还是有些高深，既然在目前非常主流的分布式和大数据领域这个Zookeeper都这么重要，那后面看来需要费一番精力来好好挖掘挖掘这项技术框架了。

## 茶余饭后
&emsp;&emsp;本段落内容摘自《从Paxos到Zookeeper》。
&emsp;&emsp;Zookeeper 最早起源于雅虎研究院的一个研究小组。在当时，研究人员发现，在雅虎内部很多大型系统基本都需要依赖一个类似的系统来进行分布式协调，但是这些系统往往都存在分布式单点问题。所以，雅虎的开发人员就试图开发一个通用的无单点问题的分布式协调框架，以便让开发人员将精力集中在处理业务逻辑上。
&emsp;&emsp;关于“ZooKeeper”这个项目的名字，其实也有一段趣闻。在立项初期，考虑到之前内部很多项目都是使用动物的名字来命名的（例如著名的Pig项目),雅虎的工程师希望给这个项目也取一个动物的名字。时任研究院的首席科学家 RaghuRamakrishnan 开玩笑地说：“在这样下去，我们这儿就变成动物园了！”此话一出，大家纷纷表示就叫动物园管理员吧一一一因为各个以动物命名的分布式组件放在一起，雅虎的整个分布式系统看上去就像一个大型的动物园了，而 Zookeeper 正好要用来进行分布式环境的协调一一于是，Zookeeper 的名字也就由此诞生了。

# 安装
> 对于一个还不那么熟悉的新框架，动手安装是最基本的入门操作之一了，也是我们可以最快接触这个框架的机会了；所以下面就通过部署集群这么一个过程来形成对Zookeeper的初步印象吧！

## 安装前准备
&emsp;&emsp;在安装前对每台主机进行命名，无论在任何时候都是值得推荐的操作，人们生来就对无意义的文字的记忆力弱于有意义的字串，所以最原始的初衷无非是让自己更分得清每个机器叫什么（就像每个人有一个名字一样）；接下来的考量，一来为了避免直接使用IP地址参与配置会导致配置文件内分不清服务器用途，二来为了在一台服务器装完直接（复制）移植到另外一台服务器只需要改动myid文件而避免每个节点都重复操作一遍，所以本次安装没有打算在配置文件内部出现IP，而是使用hostname来替代。
&emsp;&emsp;首先看看如何设置hostname，以下是在centos7中修改hostname的操作，其他系统可能略有差异。
```bash
[root@centos7 ~]$ hostnamectl set-hostname smart02
```
&emsp;&emsp;接下来，为了使集群中所有机器能通过hostname实现互通，在每台服务器的hosts文件里面都追加上下面的后三行。（其实这里第一行的配置smart01会导致zookeeper启动出现异常，具体的现象最后一个章节会详细分析）
```bash
[root@smart03 bin]# vi /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 smart01
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.211 smart01
192.168.1.212 smart02
192.168.1.213 smart03
```

## 安装规划
&emsp;&emsp;这些安装前的准备工作（包括前一小节），个人认为还是必须要的，任何时候都不建议盲目的立马就开始一件未知的事情，在开始前稍稍花一些时间简单的思考一下做个规划，总比埋头猛干要来的更合适。
&emsp;&emsp;在这次安装的demo里我打算装一个比较简单的3节点小集群；网上也有很多教程是通过伪集群来演示，资源允许的情况下，我还是觉得尽量模拟真实的集群比较合适。Zookeeper本身除了java这个依赖之外也没有别的，所以安装其实也不是很复杂。所以这次3个节点的规划如下（因为安装Zookeeper是没有主和从的说法的，所以大家都是同等级的Node）：

| Hostname | IP | 基础环境 | 节点ID | 节点定位 | 备注 |
| :--: | :--: | :--: | :--: | :--: | :--: |
| smart01 | 192.168.1.211 | jdk 1.8.0_191 | 1 | Zookeeper Node01 | 节点1 |
| smart02 | 192.168.1.212 | jdk 1.8.0_191 | 2 | Zookeeper Node02 | 节点2 |
| smart03 | 192.168.1.213 | jdk 1.8.0_191 | 3 | Zookeeper Node03 | 节点3 |

## 下载安装源
&emsp;&emsp;下载安装源最安全最可靠的去处还是官方网站，但不排除有些网站的访问很慢或者被broke了，还好下载Zookeeper的过程中没有遇到这个问题；所以我们先访问： http://zookeeper.apache.org/ ，通过官网导航找到最新Release版本的下载地址： http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz ，get到本地（get途径有很多，一般都直接wget了，网络如果不太好还是建议使用下载神器Thunder），下载完成后，就可以开始着手进行安装和配置了。

## 第一个节点的配装
&emsp;&emsp;其实在Zookeeper的装配中，不会明确指定哪个节点是Master，所有节点都是平等的，Master是在线选举的（这里就用到了Paoxs算法），所以安装的时候，除了每台机器的id不一样（id就好比节点的身份证，当然不能重复了），其他地方并没有任何差别）。
&emsp;&emsp;比较建议的装法就是先把一个节点配装调测完成，然后挨个节点分发，分发完成后直接修改id（也就是myid文件）即可，这样也避免了很多不必要的重复劳动。

### 第一步：解压
&emsp;&emsp;既然下载的是压缩包，那么第一步肯定是解压操作了；按照通常的这种组件安装的套路，下一步就是配置了，Zookeeper也提供了一个样本配置文件（zoo_sample.cfg)，但实际上Zookeeper启动默认的配置文件名是zoo.cfg，所以我们需要造出一个zoo.cfg文件，当然最简易安全的方式就是直接cp样本文件了。
```bash
[root@smart01 ins]# mkdir -p /usr/local/ins/zookeeper
[root@smart01 ins]# tar -zxvf zookeeper-3.4.13.tar.gz -C /usr/local/ins/zookeeper  --strip-components 1
......
[root@smart01 conf]# pwd
/usr/local/ins/zookeeper-3.4.13/conf
[root@smart01 conf]# ls
configuration.xsl  log4j.properties  zoo_sample.cfg
[root@smart01 conf]# cp zoo_sample.cfg zoo.cfg
[root@smart01 conf]# ls
configuration.xsl  log4j.properties  zoo.cfg  zoo_sample.cfg
[root@smart01 conf]# 
```

### 第二步：配置文件zoo.cfg
&emsp;&emsp;zoo.cfg是Zookeeper的默认启动配置（类似于nginx.conf的地位），里面包含了所有关于Zookeeper本身的一些配置项，比如：端口号、集群节点、数据目录、日志目录等等；所以这个文件是安装Zookeeper的一个核心（后面如果有必要单独开一篇讲讲这个文件）。
```bash
[root@smart01 conf]# pwd
/usr/local/ins/zookeeper/conf
[root@smart01 conf]# cat zoo.cfg 
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/usr/local/ins/zookeeper/data
# the port at which the clients will connect
clientPort=2181

# skip some info
...

# 【Lin.C】集群配置
server.1=smart01:2888:3888
server.2=smart02:2888:3888
server.3=smart03:2888:3888

[root@smart01 conf]# 
```

### 第三步：关于日志
&emsp;&emsp;相信很多同学都或多或少有遇到过组件日志过多的问题，有甚者甚至因为日志文件导致磁盘空间被挤爆导致程序挂死宕机的情况，所以永远不要忽视你认为很简单不必要关注的东西；Zookeeper的日志管理也是通过log4j.properties来操作的，关于log4j的配置之前已写过一篇专题文档介绍，这里就不多言了。
```bash
[root@smart01 conf]# pwd
/usr/local/ins/zookeeper/conf
[root@smart01 conf]# cat log4j.properties 
# Define some default values that can be overridden by system properties
# 【Lin.C】设置ROLLINGFILE，滚动日志（避免日志文件超大）
zookeeper.root.logger=INFO, ROLLINGFILE
zookeeper.console.threshold=INFO

# skip some info
...

#
# Add ROLLINGFILE to rootLogger to get log file output
#    Log DEBUG level and above messages to a log file
# 【Lin.C】每天一个文件
log4j.appender.ROLLINGFILE=org.apache.log4j.DailyRollingFileAppender
log4j.appender.ROLLINGFILE.Threshold=${zookeeper.log.threshold}
log4j.appender.ROLLINGFILE.File=${zookeeper.log.dir}/${zookeeper.log.file}

# Max log file size of 10MB
log4j.appender.ROLLINGFILE.MaxFileSize=10MB
# uncomment the next line to limit number of backup files
log4j.appender.ROLLINGFILE.MaxBackupIndex=100

# skip some info
...
[root@smart01 conf]# 
```

### 第四步：启动脚本优化
&emsp;&emsp;这一步修改是从网上别人的日志里看到的，这个zkEnv.sh我理解是跟catalina.sh类似的存在，属于启动环境初始化的一个脚本，这里修改了两个地方都是关于日志的，看来肯定是有人被这默认的日志路径和等级坑过。
```bash
[root@smart01 bin]# pwd
/usr/local/ins/zookeeper/bin
[root@smart01 bin]# cat zkEnv.sh 
#!/usr/bin/env bash

# skip some info
...

# 【Lin.C】日志路径
if [ "x${ZOO_LOG_DIR}" = "x" ]
then
    ZOO_LOG_DIR="$ZOOBINDIR/../logs"
fi

# 【Lin.C】日志打印方式
if [ "x${ZOO_LOG4J_PROP}" = "x" ]
then
    ZOO_LOG4J_PROP="INFO,ROLLINGFILE"
fi

# skip some info
...

#echo "CLASSPATH=$CLASSPATH"
[root@smart01 bin]# 
```

### 第五步：我的身份证
&emsp;&emsp;前面的步骤完成后，就要开始发身份证了，在zoo.cfg指定的数据存储目录下，需要自行创建一个myid文件，在文件里指定自己的id；这里的id也不是随意指定的，**在zoo.cfg配置文件中，配置集群时有server.x=???这种结构，这个x实际上就对应这里的myid的值。**
```bash
[root@smart01 zookeeper]# pwd
/usr/local/ins/zookeeper
[root@smart01 zookeeper]# mkdir data
[root@smart01 zookeeper]# cd data/
[root@smart01 data]# touch myid
[root@smart01 data]# echo "1" > myid 
[root@smart01 data]# cat myid 
1
[root@smart01 data]# 
```

### 第六步：单元测试
&emsp;&emsp;在日常编码中，我们经常被要求写单元测试（虽然很多时候都被我们忽视了，但实则单元测试对工程质量的重要性是非常高的，奈何国内就这样的大环境）；言归正传，装完一个节点，总要测试一下是不是能正常启动运行吧，所以这里将zoo.cfg最尾部的集群配置去除后进行了一个简单的启动测试，结果显示standalone，一切正常。
```bash
[root@smart01 ~]# cd /usr/local/ins/zookeeper/
[root@smart01 zookeeper]# cd conf/
[root@smart01 conf]# vim zoo.cfg 
[root@smart01 conf]# 
[root@smart01 conf]# cd ../bin
[root@smart01 bin]# ./zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /usr/local/ins/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@smart01 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/ins/zookeeper/bin/../conf/zoo.cfg
Mode: standalone
[root@smart01 bin]# 
```

&emsp;&emsp;在这里补充说明一下，为什么单元测试时要使用standalone模式呢？实是因为集群模式启动时，Zookeeper会尝试和其他节点建立通信，而其他节点倘若未安装或未启动，则会出现如下报错信息。（另外通过报错我们也能发现Zookeeper也是java实现的嘛！）
```bash
[root@smart01 bin]# ./zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /usr/local/ins/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@smart01 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/ins/zookeeper/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.
[root@smart01 bin]#
[root@smart01 bin]# cd ../logs/
[root@smart01 logs]# tail -200f zookeeper.log
2018-11-11 22:38:15,141 [myid:1] - WARN  [QuorumPeer[myid=1]/0:0:0:0:0:0:0:0:2181:QuorumCnxManager@584] - Cannot open channel to 3 at election address smart03/192.168.1.213:3888
java.net.NoRouteToHostException: No route to host (Host unreachable)
	at java.net.PlainSocketImpl.socketConnect(Native Method)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
	at java.net.Socket.connect(Socket.java:589)
	at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectOne(QuorumCnxManager.java:558)
	at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectAll(QuorumCnxManager.java:610)
	at org.apache.zookeeper.server.quorum.FastLeaderElection.lookForLeader(FastLeaderElection.java:838)
	at org.apache.zookeeper.server.quorum.QuorumPeer.run(QuorumPeer.java:957)
```

## 其他节点的配装
&emsp;&emsp;一旦第一个节点配装调试完成，其他节点的安装就是水到渠成的事情了，简单的做法就是直接在已装好的服务器上把安装配置好的目录打包，scp到各其他节点，解压后修改一下myid即可；所以这里针对smart02和smart03两个节点都做如下操作：
```bash
# zookeeper.ksd.tar.gz为
[root@smart01 data]# tar -czvf zookeeper.ksd.tar.gz ./zookeeper
[root@smart01 data]# scp zookeeper.ksd.tar.gz root@smart02:~
...

# 登录到smart02服务器
[root@smart02 ~]# mkdir -p /usr/local/ins/zookeeper
[root@smart02 ~]# tar -zxvf zookeeper.ksd.tar.gz   -C /usr/local/ins/zookeeper  --strip-components 2
[root@smart02 ~]# echo "2" > /usr/local/ins/zookeeper/data/myid
[root@smart02 ~]# cat /usr/local/ins/zookeeper/data/myid
2
[root@smart02 ~]# 
```

## 集群启动验证
&emsp;&emsp;所有节点安装完成后复查一遍，没有问题；于是仍然是通过 `./zkServer.sh start` 命令来启动每个节点，启动完成后通过 `./zkServer.sh status` 可以查看到节点目前的状态，最终成功启动后，应该是一个leader两个follower。
&emsp;&emsp;但是实际上往往有时候有些事与愿违的情况出现，比如我第一次搭建的时候就遇到了一个不大不小的问题......

# 集群通信的问题

## 问题初现
&emsp;&emsp;按照前面的步骤一路配置完成，挨个对3个节点分别启动后，本以为能一炮打响凯旋而归，结果一查看服务状态，发现现实和理想还是有一些些差距的...
```bash
[root@smart03 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/ins/zookeeper/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.
```
&emsp;&emsp;查看了一下日志，发现端口连接不正常，很奇怪啊，明明我的hosts都设置过了啊？而且我直接通过hostname来ping各个主机都是通的，为什么会这样呢？
```bash
2018-11-05 03:27:05,020 [myid:3] - WARN  [QuorumPeer[myid=3]/0:0:0:0:0:0:0:0:2181:QuorumCnxManager@584] - Cannot open channel to 2 at election address smart02/192.168.1.212:3888
java.net.ConnectException: Connection refused (Connection refused)
	at java.net.PlainSocketImpl.socketConnect(Native Method)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
	at java.net.Socket.connect(Socket.java:589)
	at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectOne(QuorumCnxManager.java:558)
	at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectAll(QuorumCnxManager.java:610)
	at org.apache.zookeeper.server.quorum.FastLeaderElection.lookForLeader(FastLeaderElection.java:838)
	at org.apache.zookeeper.server.quorum.QuorumPeer.run(QuorumPeer.java:957)
2018-11-05 03:27:05,021 [myid:3] - INFO  [QuorumPeer[myid=3]/0:0:0:0:0:0:0:0:2181:QuorumPeer$QuorumServer@184] - Resolved hostname: smart02 to address: smart02/192.168.1.212
2018-11-05 03:27:05,021 [myid:3] - INFO  [QuorumPeer[myid=3]/0:0:0:0:0:0:0:0:2181:FastLeaderElection@847] - Notification time out: 60000
```

## 第一次尝试
&emsp;&emsp;出现这种现象一般第一步想到的肯定是防火墙设置的问题，会不会是这家伙把通信端口拦截了？结果发现根本不是防火墙的原因，关闭防火墙之后还是涛声依旧...
```bash
#停止firewall
systemctl stop firewalld.service 
 #禁止firewall开机启动
systemctl disable firewalld.service
```

## 问题解决了...吗？
&emsp;&emsp;无奈之下只能求助万能的度娘了，果然一搜一大把的同类问题，结果发现很多博客里提到老外提供的一种解决方案：
> 把zoo.cfg里面的集群当前主机改为0.0.0.0，也就是下面这个样子，经过测试，这么一改zookeeper确实是可以正常启动了；后来又试了一下，我改成IP呢？结果也能正常启动。这就奇了怪了，**为什么要这么改就可以启动呢？？？**

```bash
# Clusters 配置法1
server.1=0.0.0.0:2888:3888
server.2=smart02:2888:3888
server.3=smart03:2888:3888

# Clusters 配置法2
server.1=192.168.1.211:2888:3888
server.2=smart02:2888:3888
server.3=smart03:2888:3888
```

## 问题真的解决了
&emsp;&emsp;突发奇想，我ping一下自己的hostname看看呢，结果发现ping的地址实际上转成了127.0.0.1，这时想起了mysql安装时有个类似的配置bind-address，很多人刚入门时想必也曾经被这个bind-address坑过。
&emsp;&emsp;手起刀落，删除各主机的hosts文件中第一行127.0.0.1到本机hostname的映射；于是，豁然开朗，柳暗花明又一村，前面的问题迎刃而解。
```bash
[root@smart01 ~]# ping smart01
PING smart01 (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.018 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.034 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.057 ms
^C
--- smart01 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
[root@smart01 ~]# vi /etc/hosts
[root@smart01 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.1.211 smart01
192.168.1.212 smart02
192.168.1.213 smart03

[root@smart01 ~]# 
[root@smart01 ~]# ping smart01
PING smart01 (192.168.1.211) 56(84) bytes of data.
64 bytes from smart01 (192.168.1.211): icmp_seq=1 ttl=64 time=0.017 ms
64 bytes from smart01 (192.168.1.211): icmp_seq=2 ttl=64 time=0.053 ms
64 bytes from smart01 (192.168.1.211): icmp_seq=3 ttl=64 time=0.056 ms
^C
--- smart01 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 0.017/0.042/0.056/0.017 ms
[root@smart01 ~]# 
```

## 风平浪静
&emsp;&emsp;重启所有zookeeper节点，查看服务状态：一个leader，两个follower；问题妥妥的解决了。
&emsp;&emsp;很多时候，我们解决问题仅仅停留在将问题解决掉的层次，并未做到知其然知其所以然，往往这种问题，下次遇到的时候，还是会束手无策，所以对待技术的钻研，在深度与广度上相较来说，鄙人更推崇的是深度优先。
```bash
[root@smart01 bin]#  ./zkServer.sh restart
ZooKeeper JMX enabled by default
Using config: /usr/local/ins/zookeeper/bin/../conf/zoo.cfg
ZooKeeper JMX enabled by default
Using config: /usr/local/ins/zookeeper/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
ZooKeeper JMX enabled by default
Using config: /usr/local/ins/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

[root@smart01 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/ins/zookeeper/bin/../conf/zoo.cfg
Mode: follower
[root@smart01 bin]# 
```