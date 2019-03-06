---
title: Kafka实战系列（二）：Springboot整合Kafka
categories:
  - 消息和缓存系列之Kafka
tags:
  - Kafka
  - Big Data
comments: true
abbrlink: 67479509
date: 2018-08-24 11:22:39
---
【引言】做Java的基本也都注意到了，Springboot最近迅速走红，当然它的走红也是个必然，所以今天也借着Springboot的东风来试试Kafka的集成；当然这个Demo也只是简单的演示一下怎么用，具体到怎么用好可不是一两个Demo可以解决的事情了。
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/public/000017.jpg" width="500">
<!-- more -->

# Maven
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

# applications.properties
```
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

# Producer Config
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

# Consumer Config
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

# Producer Service
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

# Consumer Listenser
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

# Springboot Main
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

# Test Result
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