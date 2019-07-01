---
title: Zookeeper菜鸟实录（四）：Springboot整合Zookeeper（未完成...)
categories:
  - 分布式服务框架之Zookeeper
tags:
  - Zookeeper
comments: true
abbrlink: 92eb1cd1
date: 2018-11-06 16:26:43
---
【引言】说完了那么多理论性的东西，是时候开始动手进行实际操作了。古人说得好：“纸上得来终觉浅，绝知此事要躬行”；结合时下非常流行的Springboot，我们来看看Zookeeper可以怎么玩。
<div align=center><img src="https://github.com/ttfisher/images/raw/master/public/000017.jpg" width="55%"/></div>
<!-- more -->

# 环境准备
&emsp;&emsp;硬件资源允许的情况下，我都更倾向于在Linux虚拟机上来安装各种组件的测试和Demo环境，一来更贴近于生产过程的使用（比较生产环境极少有使用windows的），二来也可以很好的跟本地系统做隔离，同时通过虚拟机快照的使用也可以很方便的做环境复原。
&emsp;&emsp;所以此次也不例外，先安装一个简单的Demo环境（这里起了个standalone的Zookeeper实例），如下：

```
[root@aliyun-db bin]# ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.214  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::20c:29ff:feda:27c0  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:da:27:c0  txqueuelen 1000  (Ethernet)
        RX packets 5327509  bytes 431605577 (411.6 MiB)
        RX errors 0  dropped 37  overruns 0  frame 0
        TX packets 127990  bytes 30825532 (29.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 1359  bytes 128928 (125.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1359  bytes 128928 (125.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@aliyun-db bin]# 
[root@aliyun-db bin]# ./zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /usr/local/ins/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@aliyun-db bin]# 
[root@aliyun-db bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/ins/zookeeper/bin/../conf/zoo.cfg
Mode: standalone
[root@aliyun-db bin]# 
```