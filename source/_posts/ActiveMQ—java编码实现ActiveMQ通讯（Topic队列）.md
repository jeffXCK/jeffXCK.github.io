---
title: ActiveMQ—java编码实现ActiveMQ通讯（Topic队列）
comments: false
date: 2019-08-31 10:38:22
categories: Active
tags: Active
---



### 发布/订阅模式消息传递的特点：

1.  生产者将消息发布到Topic中，每个消息可以有多个消费者，属于1：N的关系。
2.  生产者和消费者之间有时间上的相关性，订阅某一个主题的消费者只能消费自它订阅之后发布的消息。
3.  生产者生产时，Topic不保存消息，它是无状态的不落地的，假如无人订阅就去生产，那就是一条废消息，所以一般先启动消费者在启动生产者。
4. JMS规范允许客户创建持久订阅，这在一定程度上放松了时间上的相关性要求，持久订阅允许消费者消费它在未激活状态时发送的消息。比如微信公众号的订阅。



### 一、JMS总体架构

> ![JMS总体架构.png](https://upload-images.jianshu.io/upload_images/18660770-1d8c90a2db11afab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 二、JMS开发基本步骤:

> 1. 创建一个connection factory
>
> 2. 通过connection factory来创建 JMS connection
>
> 3. 启动JMS connection （注意勿忘：一定要启动，否则无法收到消息。）
>
> 4. 通过connection 创建JMS session
>
> 5. 创建JMS destination (queue 或者 Topic)
>
> 6. 创建JMS producer 或者创建JMS message并设置destination
> 7. 创建JMS consumer或者注册一个JMS message listener
> 8. 发送或者接收JMS message(s)
> 9. 关闭所有JMS资源（connection、session、producer、consumer等）



### 三、创建maven工程引入jar包

```java
<dependency>
  <groupId>org.apache.activemq</groupId>
  <artifactId>activemq-all</artifactId>
  <version>5.10.0</version>
</dependency>
```



### 四、编写消息发布者（producer）

```java
package com.apesbook.activemq.topic;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.util.Scanner;

/**
 * Description:
 * Author:XCK
 * Date:2019/8/31
 */
public class JmsProducer {

    public static final String BROKER_URL = "tcp://192.168.5.159:61616";
    public static final String TOPIC_NAME = "Topic.apesbook";

    public static void main(String[] args) throws JMSException {
        System.out.println("我是消息发布者");

        // 1.创建连接工厂activeMQ factory
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(BROKER_URL);
        // 2.通过连接工厂获取 connection连接, 并启动
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        // 3.通过connection连接创建 session
        Session  session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4.通过session创建目的地
        Topic topic = session.createTopic(TOPIC_NAME);
        // 5.创建生产者，并指定目的地
        MessageProducer messageProducer = session.createProducer(topic);
        // 6.发布消息
        while (true){
            // 等待键盘输入一条消息，发布到ActiveMQ，并推送给消息订阅者
            System.out.println("请输入您想发布的消息，若想退出请输入exit");
            Scanner scanner = new Scanner(System.in);
            String message = scanner.nextLine();
            if("exit".equals(message)){
                break;
            }
            TextMessage textMessage = session.createTextMessage(message);
            messageProducer.send(textMessage);
        }
        // 7.关闭资源
        messageProducer.close();
        session.close();
        connection.close();
    }
}

```



### 五、编写消息订阅者（consumer）

```java
package com.apesbook.activemq.topic;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.io.IOException;

public class JmsConsumer_messageListener {
    public static final String BROKER_URL = "tcp://192.168.5.159:61616";
    public static final String TOPIC_NAME = "Topic.apesbook";

    public static void main(String[] args) throws JMSException, IOException {
        System.out.println("我是消息订阅者1");

        // 1.创建连接工厂activeMQ factory
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(BROKER_URL);
        // 2.通过连接工厂获取 connection连接, 并启动
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        // 3.通过connection连接创建 session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4.通过session创建目的地
        Topic topic = session.createTopic(TOPIC_NAME);
        // 5.创建消费，并指定目的地
        MessageConsumer messageConsumer = session.createConsumer(topic);
        // 6.接收消息
        messageConsumer.setMessageListener(message -> {
            if(message != null && message instanceof TextMessage){
                TextMessage textMessage = (TextMessage)message;
                try {
                    System.out.println("****订阅者接收到消息：" + textMessage.getText());
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });

        System.in.read();
        // 7.关闭资源
        messageConsumer.close();
        session.close();
        connection.close();
    }
}

```

