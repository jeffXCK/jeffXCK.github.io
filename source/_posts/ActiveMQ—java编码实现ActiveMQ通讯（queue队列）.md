---
title: ActiveMQ—java编码实现ActiveMQ通讯（queue队列）
comments: false
date: 2019-08-31 10:38:22
categories: Active
tags: Active
---



### 队列模式消息传递的特点：

1.   “负载均衡模式”。
2. 生产者将消息发布到queue中，每条消息只会有一个消费者，属于1：1的关系。
3. 如果存在多个消费者，那么一条消息也只会发送给其中一个消费者，并且要求消费者ack消息。
4.  消息不会丢弃，如果当前没有消费者，queue数据默认会在MQ服务器上以文件的形式保存，ActiveMQ一般会保存在$AMQ_HOME/data/kr-store/data下面，也可以配置成DB存储。



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



### 四、编写消息生产者（producer）

```java
package com.apesbook.activemq.queue;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class JmsProducer {

    public static final String BROKER_URL = "tcp://192.168.5.159:61616";
    public static final String QUEUE_NAME = "queue01";

    public static void main(String[] args) throws JMSException {
        // 1.创建连接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(BROKER_URL);
        // 2.通过连接工厂，获得connection并启动访问
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        // 3.创建会话session
        // 两个参数，第一个叫事务/第二个叫签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4.创建目的地
        Queue queue = session.createQueue(QUEUE_NAME);
        // 5.创建消息生产者
        MessageProducer messageProducer = session.createProducer(queue);
        // 6.通过使用messageProducer生产3条消息发送到MQ队列里面
        for (int i = 1; i <= 6; i++) {
            // 7.创建消息
            TextMessage textMessage = session.createTextMessage("msg----"+i);
            // 通过messageProducer发送消息
            messageProducer.send(textMessage);
        }

        messageProducer.close();
        session.close();
        connection.close();
        System.out.println("消息发送到MQ完成");
    }
}

```

### 五、编写消息消费者（consumer）

> 1. 同步阻塞方式（receive()）

```java
package com.apesbook.activemq.queue;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.io.IOException;

public class JmsConsumer_receive {

    public static final String BROKER_URL = "tcp://192.168.5.159:61616";
    public static final String QUEUE_NAME = "queue01";

    public static void main(String[] args) throws JMSException, IOException, InterruptedException {
        System.out.println("我是1号消费者");
        // 1. 创建连接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(BROKER_URL);
        // 2.通过工厂创建connection，并启动
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        // 3. 通过工厂创建会话session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4.通过session创建目的地
        Queue queue = session.createQueue(QUEUE_NAME);
        // 5.创建消息消费者
        MessageConsumer messageConsumer = session.createConsumer(queue);

        /**
         * 同步阻塞方式（receive()）
         * 订阅者或接收者调用MessageConsumer的receive()方法来接收消息，receive()方法在能够接收到消息之前（或超时之前）将一直阻塞。
         */
        while (true) {
            // 1.等待接收消息，可以设置等待超时时间（过期不候）messageConsumer.receive(4000L)
            TextMessage textMessage = (TextMessage) messageConsumer.receive();
            if (textMessage != null) {
                if (textMessage.getText().equals("msg----3")){
                    Thread.sleep(5000);
                }
                System.out.println("****消费者接收到消息：" + textMessage.getText());
            } else {
              break;
            }
        }
        messageConsumer.close();
        session.close();
        connection.close();
    }
}

```

> 2. 异步非阻塞方式（监听器 onMessage()）:

```java
package com.apesbook.activemq.queue;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.io.IOException;

public class JmsConsumer_messageListener {

    public static final String BROKER_URL = "tcp://192.168.5.159:61616";
    public static final String QUEUE_NAME = "queue01";

    public static void main(String[] args) throws JMSException, IOException {
        System.out.println("我是2号消费者");
        // 1.创建连接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(BROKER_URL);
        // 2.通过连接工厂创建connection，并启动、并启动、并启动
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();// 启动(这一步非常关键，千万别忘记)

        // 3.通过connection创建session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4.创建目的地
        Queue queue = session.createQueue(QUEUE_NAME);
        // 5.创建消息消费者
        MessageConsumer messageConsumer = session.createConsumer(queue);

        /**
         * 通过监听的方式消费消息，
         * 异步非阻塞方式（监听器 onMessage()）
         * 订阅者或接收者调用 MessageConsumer的 setMessageListener(MessageListener messageListener) 注册一个消息监听器，
         * 当消息到达之后，系统自动调用监听器 MessageListener 的 onMessage(Message message) 方法
         */
        messageConsumer.setMessageListener(new MessageListener()
        {
            @Override
            public void onMessage(Message message) {
                if (message != null && message instanceof TextMessage){
                    TextMessage textMessage = (TextMessage) message;
                    try {
                        System.out.println("***消费者监听收到消息：" + textMessage.getText());
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        // 让程序一直保持运行
        System.in.read();
        // 关闭资源
        messageConsumer.close();
        session.close();
        connection.close();
    }
}

```

###  六、案例分析

>案例分析：
>1.先生产消息，再启动1号消费者。
>     问题：1号消费者能接收到消息吗？
>     答案：可以。
>
> 2.先生产消息，先启动1号消费者，再启动2号消费者。
>     问题：2号消费者还能消费到消息吗？
>     答案：1号可以消费到消息
>                 2号不可以消费到消息
>
> 3.先启动两个消费者，再生产6条消息。
>        问题：请问 消费情况如何？
>        答案：一人一半