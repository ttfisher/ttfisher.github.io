---
title: ActiveMQ扫盲系列（二）：点对点DEMO
categories:
  - 消息和缓存系列之ActiveMQ
tags:
  - ActiveMQ
comments: true
abbrlink: 32ba51ec
date: 2018-08-22 07:53:43
---
【引言】本章着重讲一讲ActiveMQ在点对点模式下如何实现消息传递的，包括传统方式和Springboot方式两种实现；所谓的点对点通俗理解的话，就是Producer发送的一条消息只能被监听这个Topic的某一个Consumer接收（个人理解，不见得准确）。
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2018/2018-08-21-04.jpg" width="500"/></div>
<!-- more -->

# 传统实现

## Producer
```java
package com.example.demo.traditional;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.Connection;
import javax.jms.JMSException;
import javax.jms.MessageProducer;
import javax.jms.Session;

/**
 * Created by chenglin on 2018/8/21.
 */
public class ActiveMQTProducer {

    public static final String TOPIC_NAME = "queue.traditional";

    /**
     * Have a test
     * @param args
     */
    public static void main(String[] args) {
        Connection connection = null;
        try {
            // 总体过程类似于jdbc操作，先创建连接，然后启动，然后创建目的队列，随后发送消息，然后再提交
            connection = new ActiveMQConnectionFactory(ActiveMQConnection.DEFAULT_USER,
                    ActiveMQConnection.DEFAULT_PASSWORD,
                    ActiveMQConnection.DEFAULT_BROKER_URL).createConnection();
            connection.start();
            Session session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
            MessageProducer messageProducer = session.createProducer(session.createQueue(TOPIC_NAME));
            sendMessage(session, messageProducer);
            session.commit();
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            if(connection!=null){
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 发送消息
     * @param session
     * @param messageProducer
     * @throws JMSException
     */
    public static void sendMessage(Session session, MessageProducer messageProducer) throws JMSException{
        for(int i = 0; i< 5; i++){
            messageProducer.send(session.createTextMessage("Traditional activemq send ： msg-" + i));
        }
    }
 
}
```

## Consumer
```java
package com.example.demo.traditional;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.Connection;
import javax.jms.MessageConsumer;
import javax.jms.Session;
import javax.jms.TextMessage;

/**
 * Created by chenglin on 2018/8/21.
 */
public class ActiveMQTConsumer {

    public static void main(String[] args) {
        Connection connection = null;
        try {
            connection = new ActiveMQConnectionFactory(ActiveMQConnection.DEFAULT_USER,
                    ActiveMQConnection.DEFAULT_PASSWORD,
                    ActiveMQConnection.DEFAULT_BROKER_URL).createConnection();
            connection.start();
            Session session = connection.createSession(Boolean.FALSE, Session.AUTO_ACKNOWLEDGE);
            MessageConsumer messageConsumer = session.createConsumer(session.createQueue(ActiveMQTProducer.TOPIC_NAME));
            while (true) {
                TextMessage message = (TextMessage) messageConsumer.receive(100000);
                if (message != null) {
                    System.out.println("Traditional activemq receive ：" + message.getText());
                } else {
                    break;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Result Log
```
# Consumer是阻塞等待的，下面的日志时因为运行了两遍Producer才出现的；也试过先启动Producer，此时消息会在服务端缓存，待Consumer上线之后再做消费
......
Traditional activemq receive ：Traditional activemq send ： msg-0
Traditional activemq receive ：Traditional activemq send ： msg-1
Traditional activemq receive ：Traditional activemq send ： msg-2
Traditional activemq receive ：Traditional activemq send ： msg-3
Traditional activemq receive ：Traditional activemq send ： msg-4
17:27:27.887 [ActiveMQ InactivityMonitor WriteCheckTimer] DEBUG org.apache.activemq.transport.AbstractInactivityMonitor - WriteChecker: 10001ms elapsed since last write check.
17:27:27.888 [ActiveMQ InactivityMonitor Worker] DEBUG org.apache.activemq.transport.AbstractInactivityMonitor - Running WriteCheck[tcp://127.0.0.1:61616]
Traditional activemq receive ：Traditional activemq send ： msg-0
Traditional activemq receive ：Traditional activemq send ： msg-1
Traditional activemq receive ：Traditional activemq send ： msg-2
Traditional activemq receive ：Traditional activemq send ： msg-3
Traditional activemq receive ：Traditional activemq send ： msg-4
......
```

## Result Log Plus
```
# Consumer是阻塞等待的，如果我们测试时，启动两个Consumer，然后再启动Producer来发消息，实际上消息是被两个Consumer分享的
......
# Consumer process 1
Traditional activemq receive ：Traditional activemq send ： msg-0
Traditional activemq receive ：Traditional activemq send ： msg-2
Traditional activemq receive ：Traditional activemq send ： msg-4

# Consumer process 2
Traditional activemq receive ：Traditional activemq send ： msg-1
Traditional activemq receive ：Traditional activemq send ： msg-3
......
```

# Springboot加持

## Maven引入
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

## 配置项
```xml
# URL of the ActiveMQ broker. Auto-generated by default.
# failover:(tcp://localhost:61616,tcp://localhost:61617)
# tcp://localhost:61616
spring.activemq.broker-url=tcp://localhost:61616
spring.activemq.in-memory=true

# 如果此处设置为true，需要加如下的依赖包，否则会自动配置失败，报JmsMessagingTemplate注入失败
spring.activemq.pool.enabled=false 
# 特殊依赖
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-pool</artifactId>
</dependency>
```

## FAQ
&emsp;&emsp;在使用Springboot+junit测试时，遇到两个不大不小的问题

### 报错
- java.lang.IllegalStateException: Unable to find a @SpringBootConfiguration, you need to use @ContextConfiguration or @SpringBootTest(classes=...) with your test
- Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'org.springframework.jms.core.JmsMessagingTemplate' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}

### 解决方法
- 保证测试类的包分支要和主包内的分支在同一条上（比如：主包的待测类是com.xxx.service，那么测试类也必须在com.xxx里面）；同时发现测试类的命名也有讲究，这个点没有继续深究可能理解不是很准确（也就是测试类名需要为启动类名+Tests这样）
- application.properties里面的配置项最后千万别带空格（切记）

# 单向消息传递

## Producer
```java
package com.example.demo.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.stereotype.Service;

import javax.jms.Destination;

/**
 * Created by chenglin on 2018/8/21.
 */
@Service("ActiveMQProducer")
public class ActiveMQProducer {

    // 这里也可以注入JmsTemplate，JmsMessagingTemplate是对JmsTemplate进行了封装的
    @Autowired
    private JmsMessagingTemplate jmsTemplate;

    /**
     * 发送消息
     * @param destination 消息目的队列
     * @param message 待发送消息内容
     */
    public void sendMessage(Destination destination, final String message){
        jmsTemplate.convertAndSend(destination, message);
    }
}
```

## Consumer
```java
package com.example.demo.service;

import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;

/**
 * Created by chenglin on 2018/8/21.
 */
@Component
public class ActiveMQConsumer {

    /**
     * JmsListener注解用来配置消费者监听的队列（Destination）
     * @param text 接收到的消息
     */
    @JmsListener(destination = "queue.demo")
    public void receiveQueue(String text) {
        System.out.println("This is consumer receiving ： " + text);
    }
}
```

## Tester
```java
package com.example.demo;

import com.example.demo.service.ActiveMQProducer;
import org.apache.activemq.command.ActiveMQQueue;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import javax.jms.Destination;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootdemoApplicationTests {

    @Autowired
    private ActiveMQProducer producer;

    @Test
    public void contextLoads() throws InterruptedException {
        Destination destination = new ActiveMQQueue("queue.demo");

        for(int i=0; i < 5; i++){
            System.out.println("************************************************");
            String msg = "This is chenglin speaking " + i;
            System.out.println("This is producer sending : " + msg);
            producer.sendMessage(destination, msg);
        }
    }

}
```

## Result Log
```
......
************************************************
This is producer sending : This is chenglin speaking 0
This is consumer receiving ： This is chenglin speaking 0
************************************************
This is producer sending : This is chenglin speaking 1
This is consumer receiving ： This is chenglin speaking 1
************************************************
This is producer sending : This is chenglin speaking 2
This is consumer receiving ： This is chenglin speaking 2
************************************************
This is producer sending : This is chenglin speaking 3
This is consumer receiving ： This is chenglin speaking 3
************************************************
This is producer sending : This is chenglin speaking 4
This is consumer receiving ： This is chenglin speaking 4
......
```

# 双向消息传递

## Producer
```java
package com.example.demo.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.stereotype.Service;

import javax.jms.Destination;

/**
 * Created by chenglin on 2018/8/21.
 */
@Service("ActiveMQProducer")
public class ActiveMQProducer {

    // 这里也可以注入JmsTemplate，JmsMessagingTemplate是对JmsTemplate进行了封装的
    @Autowired
    private JmsMessagingTemplate jmsTemplate;

    /**
     * 发送消息
     * @param destination 消息目的队列
     * @param message 待发送消息内容
     */
    public void sendMessage(Destination destination, final String message){
        jmsTemplate.convertAndSend(destination, message);
    }

    /**
     * 接收返回消息
     * @param text  返回的消息内容
     */
    @JmsListener(destination="queue.reply")
    public void consumerMessage(String text){
        System.out.println("This is producer receiving msg from consumer reply ： " + text);
    }

}
```

## Consumer
```java
package com.example.demo.service;

import org.springframework.jms.annotation.JmsListener;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;

import java.util.Date;
import java.util.Random;

/**
 * Created by chenglin on 2018/8/21.
 */
@Component
public class ActiveMQConsumer {

    /**
     * JmsListener注解用来配置消费者监听的队列（Destination）
     * @param text 接收到的消息
     */
    @JmsListener(destination = "queue.demo")
    @SendTo("queue.reply")
    public String receiveQueue(String text) {
        System.out.println("This is consumer receiving ： " + text);
        return "Reply from consumer : " + text;
    }
}
```

## Tester
```java
package com.example.demo;

import com.example.demo.service.ActiveMQProducer;
import org.apache.activemq.command.ActiveMQQueue;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import javax.jms.Destination;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootdemoApplicationTests {

    @Autowired
    private ActiveMQProducer producer;

    @Test
    public void contextLoads() throws InterruptedException {
        Destination destination = new ActiveMQQueue("queue.demo");

        for(int i=0; i < 5; i++){
            System.out.println("************************************************");
            String msg = "This is chenglin speaking " + i;
            System.out.println("This is producer sending : " + msg);
            producer.sendMessage(destination, msg);
        }

        // 如果这里不做一个sleep的话，返回消息的最后一条会缓存到服务端，下一次启动才会消费
        System.out.println("**********************Sleep************************");
        Thread.sleep(1000);
    }

}
```

## Result Log
```
......
# 如果上面测试类的最后不做一个sleep的话，这里就会先打印一条上一轮遗留未消费掉的最后一条消息
************************************************
This is producer sending : This is chenglin speaking 0
This is consumer receiving ： This is chenglin speaking 0
************************************************
This is producer sending : This is chenglin speaking 1
This is producer receiving msg from consumer reply ： Reply from consumer : This is chenglin speaking 0
This is consumer receiving ： This is chenglin speaking 1
************************************************
This is producer sending : This is chenglin speaking 2
This is consumer receiving ： This is chenglin speaking 2
This is producer receiving msg from consumer reply ： Reply from consumer : This is chenglin speaking 1
************************************************
This is producer sending : This is chenglin speaking 3
This is consumer receiving ： This is chenglin speaking 3
This is producer receiving msg from consumer reply ： Reply from consumer : This is chenglin speaking 2
************************************************
This is producer sending : This is chenglin speaking 4
This is consumer receiving ： This is chenglin speaking 4
This is producer receiving msg from consumer reply ： Reply from consumer : This is chenglin speaking 3
**********************Sleep************************
This is producer receiving msg from consumer reply ： Reply from consumer : This is chenglin speaking 4
......
```