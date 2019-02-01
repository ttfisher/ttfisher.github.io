---
title: Redis扫盲系列（二）：Springboot整合Redis
comments: true
categories:
  - 消息和缓存系列之Redis
tags:
  - Redis
  - Big Data
abbrlink: 65efd8ff
date: 2018-08-02 08:55:55
---
【引言】前一篇重点分析了Redis的基础概念和数据类型，以及一些常用的操作命令和特性，主要还是停留在基本的了解和操作层面，毕竟我们是做Java的，所以我们的一个核心的诉求就是通过Java怎么把Redis用起来并且还能用的很顺手，所以这一章重点来捋一捋Java中如何使用Redis。
<div align=center><img src="http://pm4hdun71.bkt.clouddn.com/img/2018/2018-08-02-03.jpg" width="500"/></div>
<!-- more -->

# Maven
&emsp;&emsp;都到了Springboot年代了，Maven自然是必不可少的，通过如下XML引入springboot的spring-boot-starter-data-redis，就可以开始开发了，So easy。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>springboottools2</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>springboottools2</name>
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
        <!-- springboot整合redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
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

# application.properties
&emsp;&emsp;Springboot的两种配置文件格式之一（properties），另外一种是yml格式（分层缩进式），配置好redis连接。
```
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=localhost
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接超时时间（毫秒）
spring.redis.timeout=60
```

# RedisOperator
&emsp;&emsp;这个类模拟了最简单的两个Redis操作：写入和读取一个String的value，作为hello world级别的试验，这个程度足矣。
```java
package com.example.demo.redis;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Repository;

import java.util.concurrent.TimeUnit;

/**
 * redis操作服务
 * @author chenglin
 */
@Repository
public class RedisOperator {

    @Autowired
    private StringRedisTemplate template;

    // 超时时间设置
    private static final int EXPIRE_SECONDS = 60;

    /**
     * 最简单的put方法
     * @param key
     * @param value
     */
    public void put(String key, String value){
        ValueOperations<String, String> ops = template.opsForValue();
        ops.set(key,value,EXPIRE_SECONDS, TimeUnit.SECONDS);
    }

    /**
     * 最简单的get方法
     * @param key
     * @return
     */
    public String get(String key){
        ValueOperations<String, String> ops = template.opsForValue();
        return ops.get(key);
    }
}
```

# Springboot Main
&emsp;&emsp;直接就上Main方法了，Springboot里面的Main方法和传统的Main方法还是有些区别的，一般创建工程时默认都会给你建好，这里比常规的Main多实现了一个CommandLineRunner，这个主要是用来满足注入和非静态调用结合到static的main里面来用才加入的。（因为静态的接口变量是没办法被Springboot直接Autowired注入的，这个具体的原因在Springboot里面再详细探讨；比如redisOperator如果被定义为static的然后直接在main里面写一些调用，是会出现NullPointerException的）
```java
package com.example.demo;

import com.example.demo.redis.RedisOperator;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.scheduling.annotation.EnableScheduling;

/**
 * 这里用到CommandLineRunner主要是为了启动时运行下面的run方法
 * @author chenglin
 */
@SpringBootApplication
@EnableScheduling
@EnableCaching
public class Springboottools2Application implements CommandLineRunner {

    public static Logger logger= LoggerFactory.getLogger(Springboottools2Application.class);

    /**
     * Springboot的启动方法，没什么好聊的
     * @param args
     */
    public static void main(String[] args) {
        SpringApplication.run(Springboottools2Application.class, args);
    }

    @Autowired
    RedisOperator redisOperator;

    @Override
    public void run(String... params) throws Exception {
        redisOperator.put("A", "My name is chenglin.");
        redisOperator.put("B", "I love my family.");
        redisOperator.put("C", "I love my friends.");
        redisOperator.put("D", "I love my work.");
        logger.info(redisOperator.get("A"));
        logger.info(redisOperator.get("B"));
        logger.info(redisOperator.get("C"));
        logger.info(redisOperator.get("D"));
    }
}
```

# Test Result
&emsp;&emsp;既然是做hello world，那自然是要看到结果输出才算完事儿的，启动Main方法后看到下面熟悉的Springboot的日志logo然后就可以静静等待结果了，不出意外的话结果都可以正常打印到Console。
```log
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.4.RELEASE)

2018-08-29 16:40:39.650  INFO 24836 --- [           main] c.e.demo.Springboottools2Application     : Starting Springboottools2Application on NB-DELL-015 with PID 24836 (started by chenglin in E:\0100-IntelliJ\9999-临时验证试用工程)
......
2018-08-29 16:40:43.648  INFO 24836 --- [           main] c.e.demo.Springboottools2Application     : My name is chenglin.
2018-08-29 16:40:43.649  INFO 24836 --- [           main] c.e.demo.Springboottools2Application     : I love my family.
2018-08-29 16:40:43.650  INFO 24836 --- [           main] c.e.demo.Springboottools2Application     : I love my friends.
2018-08-29 16:40:43.651  INFO 24836 --- [           main] c.e.demo.Springboottools2Application     : I love my work.
```

# Redis Client View
&emsp;&emsp;通过Redis的客户端我们也可以很清楚的看到，四条数据已经入库成功，证明之前的代码成功将数据写入了Redis。
<img style="clear: both;display: block;margin:auto;" src="http://pm4hdun71.bkt.clouddn.com/img/2018/2018-08-29-05.jpg" width="70%">

# More About

## 关于Template
&emsp;&emsp;初始使用springboot来整合redis的时候，其实我是忘了配置application.properties文件的，结果居然也跑出来了，后来发现springboot本身已经内置了默认配置生成的方法（猜测是已本机默认端口不带密码的模式来生成的）。
&emsp;&emsp;在上面的demo中，用到了StringRedisTemplate这个类，这个也是由redis-starter提供的生成RedisTemplate的一个实现（和Kafka、ActiveMQ类似的模式），所以就往上看了一下源码。

## StringRedisTemplate
&emsp;&emsp;StringRedisTemplate是继承自RedisTemplate（用到了泛型），想当然的以为RedisTemplate有很多不同的实现（结果看了下发现就这一个和它自己），这里的默认构造方法指定了一些属性的持久化策略，仅此而已；所以重点我们来看看RedisTemplate。
```java
public class StringRedisTemplate extends RedisTemplate<String, String> {
    public StringRedisTemplate() {
        RedisSerializer<String> stringSerializer = new StringRedisSerializer();
        this.setKeySerializer(stringSerializer);
        this.setValueSerializer(stringSerializer);
        this.setHashKeySerializer(stringSerializer);
        this.setHashValueSerializer(stringSerializer);
    }
......
```

## RedisTemplate
&emsp;&emsp;RedisTemplate源码有700多行，这里就不一行行去解读了，只针对里面看上去会影响我们使用的地方进行一些简要的说明。
&emsp;&emsp;首先我们注意到在demo里面有用到template.opsForValue()这种语法来获取操作类，果然在RedisTemplate内部还提供了一系列的操作类（从名称大致也能推测出针对不同的数据类型有不同的操作类获取方法）；这里也就先分析到这一步吧，至于RedisTemplate内部的get、set、execute、sort等等方法，实际使用时再去挖掘吧。
```java
opsForCluster
opsForGeo
opsForHash
opsForHyperLogLog
opsForList
opsForSet
opsForValue
opsForZSet
```

## 单机自动连接
&emsp;&emsp;很好奇这个单机自动连接（哪怕不配置也能操作），虽然猜测是默认写了，但是还是要看到源码才放心，所以经过一番查阅源码，最后发现了这个类JedisConnectionFactory。
```java
package org.springframework.data.redis.connection.jedis;
......
public class JedisConnectionFactory implements InitializingBean, DisposableBean, RedisConnectionFactory {
    ......
    
    // 【Lin.C】：这个配置就是默认的localhost:6379
    private RedisStandaloneConfiguration standaloneConfig = new RedisStandaloneConfiguration("localhost", Protocol.DEFAULT_PORT);
    
    ......
}

/**
 * 【Lin.C】：redis单机默认配置
 */
public class RedisStandaloneConfiguration {

    private static final String DEFAULT_HOST = "localhost";
    private static final int DEFAULT_PORT = 6379;

    private String hostName = DEFAULT_HOST;
    private int port = DEFAULT_PORT;
    private int database;
    private RedisPassword password = RedisPassword.none();
    ......
}
```

## 集群配置
&emsp;&emsp;在JedisConnectionFactory这个类里面还有两个关于集群的Configuration定义；结合下面的定义，基本也就知道配置文件里面要配置哪些项了，现在不比SpringMVC时代了，一配一大把属性，当简则简。
```java
public class JedisConnectionFactory implements InitializingBean, DisposableBean, RedisConnectionFactory {    
    // 【Lin.C】：这两个配置应该是两种不同集群模式下的配置引用
    private @Nullable RedisSentinelConfiguration sentinelConfig;
    private @Nullable RedisClusterConfiguration clusterConfig;
    ......
}

/**
 * 【Lin.C】：常用的Master-Slave集群的配置
 */
public class RedisClusterConfiguration {

    private static final String REDIS_CLUSTER_NODES_CONFIG_PROPERTY = "spring.redis.cluster.nodes";
    private static final String REDIS_CLUSTER_MAX_REDIRECTS_CONFIG_PROPERTY = "spring.redis.cluster.max-redirects";
    ......
}

/**
 * 【Lin.C】：传说中的哨兵模式的配置（虽然没咋用过，但是好像挺重要的）
 */
public class RedisSentinelConfiguration {

    private static final String REDIS_SENTINEL_MASTER_CONFIG_PROPERTY = "spring.redis.sentinel.master";
    private static final String REDIS_SENTINEL_NODES_CONFIG_PROPERTY = "spring.redis.sentinel.nodes";
    ......
}
```