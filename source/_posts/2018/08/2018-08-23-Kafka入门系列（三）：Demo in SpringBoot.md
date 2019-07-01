---
title: Kafka入门系列（三）：Demo in SpringBoot
categories:
  - 消息和缓存系列之Kafka
tags:
  - Kafka
  - Big Data
comments: true
abbrlink: 17f0c072
date: 2018-08-23 16:20:51
---
【引言】所有工具的熟悉的第一步都是从安装环境开始的，所以kafka也不例外，虽然前面在Linux服务器上已经搭建过集群了，但自己玩的话，还是在本机搭个小环境比较方便（实际上前边已经讲过Linux环境的安装方式了，这一篇会顺带说说Windows环境的安装）。
<div align=center><img src="https://github.com/ttfisher/images/raw/master/public/000017.jpg" width="55%"/></div>
<!-- more -->

# Windows环境搭建

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

## 命令模式测试

### Producer
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

### Consumer
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

### 特殊说明
&emsp;&emsp;网上很多资料都是以--zookeeper参数的方式来启动Consumer，实际上，在新版本的kafka（据说是0.90以后）已经废弃了这个--zookeeper参数了，所以这么启动会报错的（zookeeper is not a recognized option）；替代方法如下：
```
已废弃：kafka-console-consumer.bat --zookeeper localhost:2181 --topic demo
替代方法：kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic demo
```

## 小总结
&emsp;&emsp;现在类似于zookeeper、kafka这种软件基本都做到了开箱即用，配置集群也是很方便的；这里搭建这个环境主要是为了后面的Demo验证使用的，所以不追求那么完美。实际应用时肯定有很多参数调整，目录规划，日志级别和分类调整之类的操作的。

# 与SpringBoot的融合
&emsp;&emsp;做Java的基本也都注意到了，Springboot近些年迅速走红，当然它的走红也是个必然，所以今天也借着Springboot的东风来试试Kafka的集成；当然这个Demo也只是简单的演示一下怎么用，具体到怎么用好可不是一两个Demo可以解决的事情了。

## Maven的依赖引入
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>springboottools</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>springboottools</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

## Kafka配置项
```
#文件：applications.properties
#============== kafka config===================
kafka.consumer.zookeeper.connect=localhost:2181
kafka.consumer.servers=localhost:9092
kafka.consumer.enable.auto.commit=true
kafka.consumer.session.timeout=6000
kafka.consumer.auto.commit.interval=100
kafka.consumer.auto.offset.reset=latest
kafka.consumer.topic=demo
kafka.consumer.group.id=demo
kafka.consumer.concurrency=10

kafka.producer.servers=localhost:9092
kafka.producer.retries=0
kafka.producer.batch.size=4096
kafka.producer.linger=1
kafka.producer.buffer.memory=40960
```

## 生产者配置对象
```java
package com.demo.config;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;

/**
 * Producer配置对象
 * @author chenglin
 */
@Configuration
@EnableKafka
public class KafkaProducerConfig {

    // Kafka主机
    @Value("${kafka.producer.servers}")
    private String servers;

    // 消息发送失败后的重试次数
    @Value("${kafka.producer.retries}")
    private int retries;

    // kafka批量消息的大小（基于大小的batching策略）
    @Value("${kafka.producer.batch.size}")
    private int batchSize;

    // 基于时间的batching策略
    @Value("${kafka.producer.linger}")
    private int linger;

    // 缓存大小
    @Value("${kafka.producer.buffer.memory}")
    private int bufferMemory;

    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, servers);
        props.put(ProducerConfig.RETRIES_CONFIG, retries);
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, batchSize);
        props.put(ProducerConfig.LINGER_MS_CONFIG, linger);
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, bufferMemory);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return props;
    }

    public ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<String, String>(producerFactory());
    }
}
```

## 消费者配置对象
```java
package com.demo.config;

import com.demo.kafka.ConsumerListener;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;

import java.util.HashMap;
import java.util.Map;

/**
 * Consumer配置对象
 * @author chenglin
 */
@Configuration
@EnableKafka
public class KafkaConsumerConfig {

    // 连接的kafka主机
    @Value("${kafka.consumer.servers}")
    private String servers;

    // 自动提交设置
    @Value("${kafka.consumer.enable.auto.commit}")
    private boolean enableAutoCommit;

    // session超时设置
    @Value("${kafka.consumer.session.timeout}")
    private String sessionTimeout;

    // 自动提交的时间间隔
    @Value("${kafka.consumer.auto.commit.interval}")
    private String autoCommitInterval;

    // 订阅的GroupID
    @Value("${kafka.consumer.group.id}")
    private String groupId;

    // 有几种策略，控制消费的offset的：earliest、latest、none
    @Value("${kafka.consumer.auto.offset.reset}")
    private String autoOffsetReset;

    // 并发量
    @Value("${kafka.consumer.concurrency}")
    private int concurrency;

    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(concurrency);
        factory.getContainerProperties().setPollTimeout(1500);
        return factory;
    }

    public ConsumerFactory<String, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }

    public Map<String, Object> consumerConfigs() {
        Map<String, Object> propsMap = new HashMap<>();
        propsMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, servers);
        propsMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, enableAutoCommit);
        propsMap.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, autoCommitInterval);
        propsMap.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, sessionTimeout);
        propsMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        propsMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        propsMap.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        propsMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, autoOffsetReset);
        return propsMap;
    }

    /**
     * 重点：监听程序注册
     * @return
     */
    @Bean
    public ConsumerListener listener() {
        return new ConsumerListener();
    }
}
```

## 生产者服务实现
```java
package com.demo.kafka;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;

import java.util.UUID;

/**
 * ProducerService
 * @author chenglin
 */
@Component
@EnableScheduling
public class ProducerService {

    protected final Logger logger = LoggerFactory.getLogger(this.getClass());

    // 消息发送的主题
    public static String TOPIC_NAME = "demo";

    // 消息的key
    public static String TOPIC_KEY = "ttfisher";

    // kafka发送消息的一个对象（很像前几天试验ActiveMQ的那种模式）
    @Autowired
    private KafkaTemplate kafkaTemplate;

    /**
     * 这里使用了一个循环调度的方式，每3秒钟发送一条消息到kafka
     */
    @Scheduled(cron = "00/3 * * * * ?")
    public void send() {
        try {
            String msg = UUID.randomUUID().toString();
            logger.info("Kafka producer send msg : " + TOPIC_KEY + " --> " + msg);
            kafkaTemplate.send(TOPIC_NAME, TOPIC_KEY, msg);
        } catch (Exception e) {
            logger.error("Kafka send msg failure : ", e);
        }
    }
}
```

## 消费者监听实现
```java
package com.demo.kafka;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;

/**
 * Consumer监听
 * @author chenglin
 */
public class ConsumerListener {

    protected final Logger logger = LoggerFactory.getLogger(this.getClass());

    /**
     * 主要是通过这个注解完成消息订阅的（topics指定订阅的主题）
     * @param record 接收到的消息，是一个<K, V>对象
     */
    @KafkaListener(topics = {"demo"})
    public void listen(ConsumerRecord<?, ?> record) {
        logger.info("Kafka consumer received msg : " + record.key() + " --> " + record.value().toString());
    }
}
```

# 本地模拟测试

## 测试启动方法
```java
package com.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringboottoolsApplication {

    /**
     * Springboot的启动方法，没什么好聊的
     * @param args
     */
    public static void main(String[] args) {
        SpringApplication.run(SpringboottoolsApplication.class, args);
    }
}
```

## 测试结果日志
```
......
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.4.RELEASE)
......
2018-08-27 14:00:34.992  INFO 20824 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-1, groupId=demo] Setting newly assigned partitions [demo-0]
2018-08-27 14:00:34.993  INFO 20824 --- [ntainer#0-2-C-1] o.a.k.c.c.internals.AbstractCoordinator  : [Consumer clientId=consumer-3, groupId=demo] Successfully joined group with generation 7
2018-08-27 14:00:34.994  INFO 20824 --- [ntainer#0-2-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-3, groupId=demo] Setting newly assigned partitions []
2018-08-27 14:00:34.994  INFO 20824 --- [ntainer#0-2-C-1] o.s.k.l.KafkaMessageListenerContainer    : partitions assigned: []
2018-08-27 14:00:34.995  INFO 20824 --- [ntainer#0-0-C-1] o.s.k.l.KafkaMessageListenerContainer    : partitions assigned: [demo-0]
2018-08-27 14:00:34.996  INFO 20824 --- [ntainer#0-7-C-1] o.a.k.c.c.internals.AbstractCoordinator  : [Consumer clientId=consumer-8, groupId=demo] Successfully joined group with generation 7
2018-08-27 14:00:34.996  INFO 20824 --- [ntainer#0-7-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-8, groupId=demo] Setting newly assigned partitions []
2018-08-27 14:00:34.996  INFO 20824 --- [ntainer#0-7-C-1] o.s.k.l.KafkaMessageListenerContainer    : partitions assigned: []
2018-08-27 14:00:36.001  INFO 20824 --- [pool-1-thread-1] com.demo.kafka.ProducerService           : Kafka producer send msg : ttfisher --> 2896033b-c365-4b05-acf5-b42c682040be
2018-08-27 14:00:36.011  INFO 20824 --- [ntainer#0-0-C-1] com.demo.kafka.ConsumerListener          : Kafka consumer received msg : ttfisher --> 2896033b-c365-4b05-acf5-b42c682040be
2018-08-27 14:00:39.003  INFO 20824 --- [pool-1-thread-1] com.demo.kafka.ProducerService           : Kafka producer send msg : ttfisher --> f28512b3-2479-4d5a-b0e2-e4afd7d07f93
2018-08-27 14:00:39.010  INFO 20824 --- [ntainer#0-0-C-1] com.demo.kafka.ConsumerListener          : Kafka consumer received msg : ttfisher --> f28512b3-2479-4d5a-b0e2-e4afd7d07f93
2018-08-27 14:00:42.002  INFO 20824 --- [pool-1-thread-1] com.demo.kafka.ProducerService           : Kafka producer send msg : ttfisher --> 194cfa96-22c2-4e3c-bc7d-36791b545ec8
2018-08-27 14:00:42.008  INFO 20824 --- [ntainer#0-0-C-1] com.demo.kafka.ConsumerListener          : Kafka consumer received msg : ttfisher --> 194cfa96-22c2-4e3c-bc7d-36791b545ec8
2018-08-27 14:00:45.001  INFO 20824 --- [pool-1-thread-1] com.demo.kafka.ProducerService           : Kafka producer send msg : ttfisher --> c80b05a1-e372-4444-9c6b-0a42b451f6ee
2018-08-27 14:00:45.009  INFO 20824 --- [ntainer#0-0-C-1] com.demo.kafka.ConsumerListener          : Kafka consumer received msg : ttfisher --> c80b05a1-e372-4444-9c6b-0a42b451f6ee
2018-08-27 14:00:48.002  INFO 20824 --- [pool-1-thread-1] com.demo.kafka.ProducerService           : Kafka producer send msg : ttfisher --> 14e24322-d441-49d4-bdcc-a0da555f87cb
2018-08-27 14:00:48.011  INFO 20824 --- [ntainer#0-0-C-1] com.demo.kafka.ConsumerListener          : Kafka consumer received msg : ttfisher --> 14e24322-d441-49d4-bdcc-a0da555f87cb
......
```