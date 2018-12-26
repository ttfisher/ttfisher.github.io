---
title: ActiveMQ实践系列（一）：基于JMX的服务监控
categories:
  - 消息和缓存系列之ActiveMQ
tags:
  - ActiveMQ
comments: true
abbrlink: b4afbbb6
date: 2018-12-17 15:09:11
---
【引言】无意间接触到一种消息积压处理的场景，想着是否可以监控MQ的队列参数进行发送端的策略调整，于是有了这一篇文章；当然本次也参考了网络上很多的分享，但无奈的发现天下文章一大抄，很难找到一篇写的完整的。
<div align=center><img src="/img/public/000019.jpg" width="500"/></div>
<!-- more -->

# 前言
&emsp;&emsp;这次研究这个主题的起因是因为

# 基础环境

## 安装Activemq
&emsp;&emsp;鉴于本机并没有AMQ的环境，于是临时下载了一个Release版本安装了一下（其实也谈不上安装那么复杂，解压即用而已），AMQ的目录结构大致是下面这个样子（有点类似于tomcat的目录结构），本次重点涉及的几个文件都做了标注，以备参考：
```
PS D:\05-Pracs\apache-activemq-5.15.8> tree /f
文件夹 PATH 列表
卷序列号为 DA18-EBFA
D:.
│
├─bin
│  │  activemq                 # 启动配置文件
│  │  activemq.bat             # 启动文件（实际使用时可能用到bin下面的32和64两个子目录下的activemq.bat直接启动）
│                              # 使用这里的启动文件，需要全命令模式（.\activemq.bat start）
├─conf
│      activemq.xml            # activemq自身的配置文件
│      ...
│      jmx.access              # 用户管理
│      jmx.password            # 用户密码管理
│      ...
│
├─data
│  ├─...
│
├─docs
│  ├─...
│
├─examples
│  ├─...
│
├─lib
│  ├─...
│
├─webapps
│  ├─...
│
├─webapps-demo
│  ├─...

PS D:\05-Pracs\apache-activemq-5.15.8>
```

## 修改配置项
&emsp;&emsp;配置项的修改其实比较简单，也就是在managementContext节点开启一个端口（本项修改涉及的文件为conf目录下的activemq.xml）：
```xml
<!-- 修改前 -->
<managementContext>
    <managementContext/>
</managementContext>

<!-- 修改后 -->
<managementContext>
    <managementContext createConnector="true" connectorPort="11099" />
</managementContext>
```

## 添加新用户
&emsp;&emsp;这里的配置主要是控制登录用户的连接后操作权限的（本项修改涉及的文件为conf目录下的jmx.access和jmx.password两个文件）：
```
# jmx.access文件，定义用户的权限
admin readwrite
monitor readonly

# jmx.password文件，定义用户登录密码
admin activemq
monitor smart
```

## 修改启动项
&emsp;&emsp;这里添加了一些启动参数，其实也是基于前面的几项操作对应添加的，比如端口号、用户文件、密码文件等等（本项修改涉及的文件为bin目录下的activemq）；修改位置在invoke_start方法之前：
```
# - $ACTIVEMQ_SSL_OPTS     : options for SSL encryption

ACTIVEMQ_CONF="D:/05-Pracs/apache-activemq-5.15.8/conf"
ACTIVEMQ_SUNJMX_START="-Dcom.sun.management.jmxremote.port=11099 "
ACTIVEMQ_SUNJMX_START="$ACTIVEMQ_SUNJMX_START -Dcom.sun.management.jmxremote.password.file=${ACTIVEMQ_CONF}/jmx.password"
ACTIVEMQ_SUNJMX_START="$ACTIVEMQ_SUNJMX_START -Dcom.sun.management.jmxremote.access.file=${ACTIVEMQ_CONF}/jmx.access"
ACTIVEMQ_SUNJMX_START="$ACTIVEMQ_SUNJMX_START -Dcom.sun.management.jmxremote.ssl=false"

invoke_start(){
```

## 启动Activemq
&emsp;&emsp;通过bat脚本启动AMQ服务，通过观察日志可以发现很多有用信息（比如：JMX的启动信息包括启动端口号，不同的连接服务信息比如TCP、Stomp等等）：
```
PS D:\05-Pracs\apache-activemq-5.15.8\bin> .\activemq.bat start
Java Runtime: Oracle Corporation 1.8.0_181 C:\Program Files\Java\jdk1.8.0_181\jre
  Heap sizes: current=1005056k  free=989327k  max=1005056k
    JVM args: -Dcom.sun.management.jmxremote -Xms1G -Xmx1G -Djava.util.logging.config.file=logging.properties ...
Extensions classpath:
  [D:\05-Pracs\apache-activemq-5.15.8\bin\..\lib,...]
ACTIVEMQ_HOME: D:\05-Pracs\apache-activemq-5.15.8\bin\..
ACTIVEMQ_BASE: D:\05-Pracs\apache-activemq-5.15.8\bin\..
ACTIVEMQ_CONF: D:\05-Pracs\apache-activemq-5.15.8\bin\..\conf
ACTIVEMQ_DATA: D:\05-Pracs\apache-activemq-5.15.8\bin\..\data
Loading message broker from: xbean:activemq.xml
 INFO | Refreshing org.apache.activemq.xbean.XBeanBrokerFactory$1@5f2108b5: startup date [Tue Dec 18 15:13:33 CST 2018];...
 INFO | Using Persistence Adapter: KahaDBPersistenceAdapter[D:\05-Pracs\apache-activemq-5.15.8\bin\..\data\kahadb]
 INFO | JMX consoles can connect to service:jmx:rmi:///jndi/rmi://localhost:11099/jmxrmi
 INFO | KahaDB is version 6
 INFO | PListStore:[D:\05-Pracs\apache-activemq-5.15.8\bin\..\data\localhost\tmp_storage] started
 INFO | Apache ActiveMQ 5.15.8 (localhost, ID:IVF21BBAA9XWLKD-65387-1545117214542-0:1) is starting
 INFO | Listening for connections at: 
    tcp://IVF21BBAA9XWLKD:61616?maximumConnections=1000&wireFormat.maxFrameSize=104857600
 INFO | Connector openwire started
 INFO | Listening for connections at: 
    amqp://IVF21BBAA9XWLKD:5672?maximumConnections=1000&wireFormat.maxFrameSize=104857600
 INFO | Connector amqp started
 INFO | Listening for connections at: 
    stomp://IVF21BBAA9XWLKD:61613?maximumConnections=1000&wireFormat.maxFrameSize=104857600
 INFO | Connector stomp started
 INFO | Listening for connections at: 
    mqtt://IVF21BBAA9XWLKD:1883?maximumConnections=1000&wireFormat.maxFrameSize=104857600
 INFO | Connector mqtt started
 INFO | Starting Jetty server
 INFO | Creating Jetty connector
 WARN | ServletContext@o.e.j.s.ServletContextHandler@38604b81{/,null,STARTING} has uncovered http methods for path: /
 INFO | Listening for connections at 
    ws://IVF21BBAA9XWLKD:61614?maximumConnections=1000&wireFormat.maxFrameSize=104857600
 INFO | Connector ws started
 INFO | Apache ActiveMQ 5.15.8 (localhost, ID:IVF21BBAA9XWLKD-65387-1545117214542-0:1) started
 INFO | For help or more information please see: http://activemq.apache.org
 INFO | No Spring WebApplicationInitializer types detected on classpath
 INFO | ActiveMQ WebConsole available at http://0.0.0.0:8161/
 INFO | ActiveMQ Jolokia REST API available at http://0.0.0.0:8161/api/jolokia/
 INFO | Initializing Spring FrameworkServlet 'dispatcher'
 INFO | No Spring WebApplicationInitializer types detected on classpath
 INFO | jolokia-agent: Using policy access restrictor classpath:/jolokia-access.xml
 WARN | Transport Connection to: tcp://127.0.0.1:60850 failed: java.net.SocketException: Connection reset
 WARN | Transport Connection to: tcp://127.0.0.1:57564 failed: java.net.SocketException: Connection reset
```

# 编码测试

## Maven引入
&emsp;&emsp;本次编码涉及到的关于activemq-web的一些类，需要通过Maven引入：
```xml
<!-- https://mvnrepository.com/artifact/org.apache.activemq/activemq-web -->
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-web</artifactId>
    <version>5.15.8</version>
</dependency>
```

## Demo Code
&emsp;&emsp;这里的Demo也没有过多的操作，只是针对AMQ的一些基础信息做了常规的打印，有扩展需求的话可以根据这里的打印信息参考扩展：
```java
package com.ttfisher;

import org.apache.activemq.broker.jmx.BrokerViewMBean;
import org.apache.activemq.broker.jmx.QueueViewMBean;
import org.apache.activemq.broker.jmx.TopicViewMBean;
import org.apache.activemq.web.RemoteJMXBrokerFacade;
import org.apache.activemq.web.config.SystemPropertiesConfiguration;

/**
 * 基于JMX的Activemq服务状态监控
 *
 * @author: chenglin
 * @date: 2018/12/18 18:59
 */
public class JMXMonitor {

    private static String jmxUrl = "service:jmx:rmi:///jndi/rmi://127.0.0.1:11099/jmxrmi";
    private static String jmxUser = "monitor";
    private static String jmxPwd = "smart";

    /**
     * 监控服务
     */
    private static void doMonitor() {
        try {
            // 创建连接
            RemoteJMXBrokerFacade createConnector = new RemoteJMXBrokerFacade();
            System.setProperty("webconsole.jmx.url", jmxUrl);
            System.setProperty("webconsole.jmx.user", jmxUser);
            System.setProperty("webconsole.jmx.password", jmxPwd);
            createConnector.setConfiguration(new SystemPropertiesConfiguration());

            // 监控Broker
            BrokerViewMBean brokerAdmin = createConnector.getBrokerAdmin();
            System.out.println("============= Broker Info ================");
            System.out.println("BrokerName: " + brokerAdmin.getBrokerName());
            System.out.println("TotalMessageCount: " + brokerAdmin.getTotalMessageCount());
            System.out.println("TotalConsumerCount: " + brokerAdmin.getTotalConsumerCount());
            System.out.println("TotalDequeueCount: " + brokerAdmin.getTotalDequeueCount());
            System.out.println("TotalEnqueueCount: " + brokerAdmin.getTotalEnqueueCount());

            // 监控所有的Topic
            System.out.println("============= Topic List Info ================");
            for (TopicViewMBean topicViewMBean : createConnector.getTopics()) {
                System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
                System.out.println("Topic Name: " + topicViewMBean.getName());
                System.out.println("ConsumerCount: " + topicViewMBean.getConsumerCount());
                System.out.println("DequeueCount: " + topicViewMBean.getDequeueCount());
                System.out.println("EnqueueCount: " + topicViewMBean.getEnqueueCount());
                System.out.println("DispatchCount: " + topicViewMBean.getDispatchCount());
                System.out.println("ExpiredCount: " + topicViewMBean.getExpiredCount());
                System.out.println("MaxEnqueueTime: " + topicViewMBean.getMaxEnqueueTime());
                System.out.println("ProducerCount: " + topicViewMBean.getProducerCount());
                System.out.println("MemoryPercentUsage: " + topicViewMBean.getMemoryPercentUsage());
                System.out.println("MemoryLimit: " + topicViewMBean.getMemoryLimit());
            }

            // 监控所有的Queue
            System.out.println("============ Queue List Info ==================");
            for (QueueViewMBean queueViewMBean : createConnector.getQueues()) {
                System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
                System.out.println("Queue Name: " + queueViewMBean.getName());
                System.out.println("ConsumerCount: " + queueViewMBean.getConsumerCount());
                System.out.println("DequeueCount: " + queueViewMBean.getDequeueCount());
                System.out.println("EnqueueCount: " + queueViewMBean.getEnqueueCount());
                System.out.println("DispatchCount: " + queueViewMBean.getDispatchCount());
                System.out.println("ExpiredCount: " + queueViewMBean.getExpiredCount());
                System.out.println("MaxEnqueueTime: " + queueViewMBean.getMaxEnqueueTime());
                System.out.println("ProducerCount: " + queueViewMBean.getProducerCount());
                System.out.println("MemoryPercentUsage: " + queueViewMBean.getMemoryPercentUsage());
                System.out.println("MemoryLimit: " + queueViewMBean.getMemoryLimit());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * Have a test
     *
     * @param args
     */
    public static void main(String[] args) {
        doMonitor();
    }
}
```

## 测试结果
&emsp;&emsp;通过测试日志能看到AMQ的基本信息和实时的数据状态，比如Broker的信息，不同Topic的信息（包括出入队的数据量等）：
```
08:41:46.963 [main] DEBUG org.apache.activemq.web.RemoteJMXBrokerFacade - Creating a new JMX-Connection to the broker
08:41:47.219 [main] INFO org.apache.activemq.web.RemoteJMXBrokerFacade - Connected via JMX to the broker at service:jmx:rmi:///jndi/rmi://127.0.0.1:11099/jmxrmi
============= Broker Info ================
BrokerName: localhost
TotalMessageCount: 0
TotalConsumerCount: 0
TotalDequeueCount: 55
TotalEnqueueCount: 610
============= Topic List Info ================
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Topic Name: queue.pub.sub
ConsumerCount: 0
DequeueCount: 55
EnqueueCount: 120
DispatchCount: 55
ExpiredCount: 0
MaxEnqueueTime: 6
ProducerCount: 0
MemoryPercentUsage: 0
MemoryLimit: 720424141
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Topic Name: ActiveMQ.Advisory.MasterBroker
ConsumerCount: 0
DequeueCount: 0
EnqueueCount: 1
DispatchCount: 0
ExpiredCount: 0
MaxEnqueueTime: 0
ProducerCount: 0
MemoryPercentUsage: 0
MemoryLimit: 720424141
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Topic Name: ActiveMQ.Advisory.Connection
ConsumerCount: 0
DequeueCount: 0
EnqueueCount: 244
DispatchCount: 0
ExpiredCount: 0
MaxEnqueueTime: 0
ProducerCount: 0
MemoryPercentUsage: 0
MemoryLimit: 720424141
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Topic Name: ActiveMQ.Advisory.Consumer.Topic.queue.pub.sub
ConsumerCount: 0
DequeueCount: 0
EnqueueCount: 4
DispatchCount: 0
ExpiredCount: 0
MaxEnqueueTime: 0
ProducerCount: 0
MemoryPercentUsage: 0
MemoryLimit: 720424141
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Topic Name: ActiveMQ.Advisory.Topic
ConsumerCount: 0
DequeueCount: 0
EnqueueCount: 1
DispatchCount: 0
ExpiredCount: 0
MaxEnqueueTime: 0
ProducerCount: 0
MemoryPercentUsage: 0
MemoryLimit: 720424141
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Topic Name: ActiveMQ.Advisory.Producer.Topic.queue.pub.sub
ConsumerCount: 0
DequeueCount: 0
EnqueueCount: 240
DispatchCount: 0
ExpiredCount: 0
MaxEnqueueTime: 0
ProducerCount: 0
MemoryPercentUsage: 0
MemoryLimit: 720424141
============ Queue List Info ==================

Process finished with exit code 0
```