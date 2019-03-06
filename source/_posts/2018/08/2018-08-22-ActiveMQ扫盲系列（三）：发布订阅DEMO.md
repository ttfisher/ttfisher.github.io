---
title: ActiveMQ扫盲系列（三）：发布订阅DEMO
categories:
  - 消息和缓存系列之ActiveMQ
tags:
  - ActiveMQ
comments: true
abbrlink: d59227c
date: 2018-08-22 08:48:43
---
【引言】本章着重讲一讲ActiveMQ在发布订阅模式下如何实现消息传递的，本章只写了传统方式实现，Springboot的方式前面已经试验过了，就不再尝试了；所谓的发布订阅通俗理解的话，就是Producer发送的一条消息能被所有监听这个Topic的Consumer接收，人手一份（个人理解，不见得准确）。
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2018/2018-08-21-05.jpg" width="500"/></div>
<!-- more -->

# Producer
```java
package com.example.demo.pubsub;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * Created by chenglin on 2018/8/22.
 * @author chenglin
 */
public class PubTopicProducer {

    public static final String TOPIC_NAME = "queue.pub.sub";

    /**
     * 发送消息
     * @param msg
     */
    public void send(String msg) {
        Connection connection = null;
        try {
            // 创建一个connection，并打开连接
            connection = new ActiveMQConnectionFactory().createConnection();
            connection.start();
 
            Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
            MessageProducer producer = session.createProducer(session.createTopic(TOPIC_NAME));
 
            // 消息是否持久化
            producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
            producer.send(session.createTextMessage(msg));
        } catch (JMSException e) {
            e.printStackTrace();
        } finally{
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
     * Have a test
     * @param args
     */
    public static void main(String[] args) {
        PubTopicProducer producer = new PubTopicProducer();
        for (int i = 0; i < 5; i++) {
            producer.send("This is pub sending msg : msg-" + i);
        }
    }
 
}
```

# Consumer
```java
package com.example.demo.pubsub;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * Created by chenglin on 2018/8/22.
 * @author chenglin
 */
public class SubTopicConsumer implements MessageListener {
 
    @Override
    public void onMessage(Message message) {
        if (message instanceof TextMessage) {
            TextMessage txtMsg = (TextMessage) message;
            try {
                System.out.println("This is sub receiving msg : " + txtMsg.getText());
            } catch (JMSException e) {
                e.printStackTrace();
            }
 
        }
    }

    /**
     * 消息接收流程
     */
    public void receive() {
        Connection connection = null;
 
        try {
            connection = new ActiveMQConnectionFactory().createConnection();
            connection.start();
            Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
            MessageConsumer consumer = session.createConsumer(session.createTopic(PubTopicProducer.TOPIC_NAME));
            consumer.setMessageListener(new SubTopicConsumer());
        } catch (JMSException e) {
            e.printStackTrace();
        }/* 这里为什么不能关闭connection？一旦关闭了，监听就会断了，就无法接收消息了
        finally{
            if(connection!=null){
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }*/
    }

    /**
     * Have a test
     * @param args
     */
    public static void main(String[] args) {
        new SubTopicConsumer().receive();
    }
 
}

```

# Result Log
```
# Producer发送了5条消息

# Consumer Process 1
This is sub receiving msg : This is pub sending msg : msg-0
This is sub receiving msg : This is pub sending msg : msg-1
This is sub receiving msg : This is pub sending msg : msg-2
This is sub receiving msg : This is pub sending msg : msg-3
This is sub receiving msg : This is pub sending msg : msg-4

# Consumer Process 2
This is sub receiving msg : This is pub sending msg : msg-0
This is sub receiving msg : This is pub sending msg : msg-1
This is sub receiving msg : This is pub sending msg : msg-2
This is sub receiving msg : This is pub sending msg : msg-3
This is sub receiving msg : This is pub sending msg : msg-4
```

# 消息流程总结
&emsp;&emsp;下图是从网上看到的一张流程总结，先致谢一下！图画的很清晰，一目了然；整个流程围绕创建连接-创建会话-发送/接收消息-释放连接的模式展开，非常类似于jdbc操作的流程。
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-08-21-07.jpg" width="50%">