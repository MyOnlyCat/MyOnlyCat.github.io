---
layout: post
title: RabbitMQ笔记-3
categories: RabbitMQ
tags:
  - RabbitMQ
abbrlink: ceab54f3
date: 2019-01-24 00:00:00
---

**这里记录官方的第2个demo,这个demo中主要讲的MQ的队列和消费者在处理消息的同时挂掉了的情况消息怎么去处理,以及任务的公平调度**
<!-- more -->

[TOC]

### 0-概要

* 在官方的第一个Demo中只有一个消费者,而且任务简单并没有在真实情况下进行模拟,比如遇到处理复杂的任务需要处理几秒钟的那种,在这个Demo中将使用Thread.sleep()进行模拟复杂任务的等待情况,并且启动第二个消费者

### 1-循环调度简单实现

* 任务队列是要是避免立即执行复杂型任务,并且立即执行,我们需要把它封装成一个任务放到任务队列.后台将执行任务.当运行多个工作进程(消费者)时它们之间可以共享任务

* 循环调度的有点就是任务可以并行执行,假如任务数量巨大,我们可以添加更多的消费者,扩展起来非常轻松

* 如下图

![3531efe79d7b6c8e1f013e7eb3b0b600.png](https://note.youdao.com/yws/api/personal/file/28CB2C088C274940B491244293070FFA?method=download&shareKey=ddc494a317297cd865e975d567948230)


#### 1.1-生产者Java端编写

```java

package demo02;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

/**
 * 官方Demo 02 生产者
 */
public class NewTask {

    /**
     * 定义队列名称
     */
    private final static String QUEUE_NAME = "task_queue";

    public static void main(String[] args) {

        //创建一个连接
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("user");
        factory.setPassword("123456");

        //注意这里使用了 try-with-resources,
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            //普通队列声明
            //参数的大概意思 参数1 queue : 队列名称
            //             参数2 Durable : 是否持久化队列(该队列在服务器重启过后存活)
            //             参数3 exclusive : 是否声明一个排他性队列(以后在写文章进行统一解释MQ的队列类型)
            //             参数4 autoDelete : 是否声明一个自动删除的队列
            //             参数5 arguments : 队列的其它属性(构造参数)
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);

            /**
             * 持久化队列声明,参数 durable 为 true
             * channel.queueDeclare(QUEUE_NAME, true, false, false, null);
             */

            String message = "Hello My Name Is Task 01 ";
            //普通消息声明
            //  参数1 exchange : 交换机名称,默认的话是空字符串,消息通过routingKey指定的名称路由到队列（如果存在,是直接路由到队列）
            //  参数2 routingKey : 路由的key 我理解为指定消息发送到哪个队列
            //  参数3 BasicProperties : 可以自行查看BasicProperties类的定义
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));

            /**
             * 持久化消息声明,修改了BasicProperties参数
             * channel.basicPublish("", QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes(StandardCharsets.UTF_8))
             */
            System.out.println(" [x] Sent '" + message + "'");
        } catch (TimeoutException | IOException e) {
            e.printStackTrace();
        }

    }

}


```

#### 1.2-消费者Java端编写

```java

package demo02;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;
import java.util.concurrent.TimeoutException;

/**
 * 官方Demo 02 消费者
 */
public class Work {

    /**
     * 队列名称,保持与send端一样
     */
    private final static String QUEUE_NAME = "task_queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        //连接
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("user");
        factory.setPassword("123456");

        //为什么这里不使用 try-with-resource语句来自动关闭通道和连接？
        //因为在生产者中给 队列发送消息过后关闭连接没有什么影响
        //但是在 消费者中,需要异步监听,当消息到达时需要保持进程在活跃中
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        //队列声明
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        //它将缓冲消息,直到准备使用它,这就是DeliverCallback子类的作用
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" + message + "'");
            try {
                doWork(message);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println(" [X] Done My Num is 01");
            }
        };
        boolean autoAck = true;
        channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, consumerTag -> {
        });

    }

    /**
     * 模拟处理耗时任务
     *
     * @param task
     * @throws InterruptedException
     */
    private static void doWork(String task) throws InterruptedException {

        Thread.sleep(10000);

    }

}


```

#### 1.3-效果展示

* 我们复制一份**Work.java**的代码,把它当做第二个消费者去共同处理任务,生产者发送消息过后,第一个被调度的消费者处理消息需要等待10秒,再次发送消息不会发送到第一个消费者去,而是发送到第二个消费者去处理

* 效果如下图

![0e741fd5d9d7284e6b0d6c9c46457032.png](https://note.youdao.com/yws/api/personal/file/4E015608EA684A73AFDAC1ECBD25E283?method=download&shareKey=d319dd3bb7535b1760caa2e116ce1a19)

* * *

![81b64699f555c6ebe7b6785d9943a8c7.png](https://note.youdao.com/yws/api/personal/file/F62995C472B34F2C8BACE00E0BBDBE71?method=download&shareKey=a9a6713e81fae4f5bc45ce844e4d9c87)

* * *

* 可以看出任务是被轮训调度给消费者的


### 2-消息确认


* 当任务调度到消费者过后,开始处理任务的过程当中消费者因为各种原因下线了会发生什么.如果使用上面的Demo进行测试的话,一旦MQ向消费者发送了消息,消息将立刻标记为删除,在这种情况下如果一个消费者在处理任务的过程中下线了,会丢失刚刚处理的任务,还将丢失发送给这个消费者但是还没开始处理的所有消息

* * *

* 我们希望的是当消费者意外下线的情况发送,任务将会交给另外一个消费者进行处理 

* * *

* 为了保证消息永不丢失,MQ支持 **message acknowledgments(消息确认)**,消费者发回**ack(nowledgement)** 告诉MQ已收到,处理了消息MQ可以自由删除它

* * *

* 如果消费者意外下线,**channel(通道)关闭, connection(连接)关闭,或者TCP连接丢失** 而没有发送确认,MQ将理解为消息未完成处理并且将重新放入队列中.如果同时还有其它消费者在线,则会迅速将其发送给其它消费者.这样就可以确保即使消费者下线,消息也不会丢失

* * *

* 在默认情况下,**Manual message acknowledgments(手动消息确认)** 是已经打开的,但是在上面的代码中我们已经关闭了它,使用的**autoAck=true** ,在真实情况下一旦我们完成的任务,就应该将此标志设置为**false**并从消费者发送确认

* * *

#### 2.1-消息确认Java消费者端代码

* 修改过的消费者代码如下

```java

package demo02;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

public class Work02 {

    /**
     * 队列名称,保持与send端一样
     */
    private final static String QUEUE_NAME = "task_queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        //连接
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("182.61.24.136");
        factory.setUsername("demoUser");
        factory.setPassword("Yilunjk123");

        //为什么这里不使用 try-with-resource语句来自动关闭通道和连接？
        //因为在生产者中给 队列发送消息过后关闭连接没有什么影响
        //但是在 消费者中,需要异步监听,当消息到达时需要保持进程在活跃中
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        //队列声明
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        //它将缓冲消息,直到准备使用它,这就是DeliverCallback子类的作用
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" + message + "'");
            try {
                doWork(message);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println(" [X] Done My Num is 02");
                //添加的消息确认
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };
        //关闭了自动确认
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, consumerTag -> {
        });

    }

    /**
     * 模拟处理耗时任务
     *
     * @param task
     * @throws InterruptedException
     */
    private static void doWork(String task) throws InterruptedException {
        Thread.sleep(10000);
    }

}


```


* * *

#### 2.2-效果演示

* 生产者发送消息

![0a74e02e1aeed6cda4219fe6a6ba752a.png](https://note.youdao.com/yws/api/personal/file/26DF701973384DE0A4DE22736011B605?method=download&shareKey=346f7c470da4789e55d2a60fcfb39a93)

* * *

* 消费者1接收到消息,但是处理过程中下线

![ebe4eda16c3975b0f48621dfb8cc4a7a.png](https://note.youdao.com/yws/api/personal/file/0C45BAAB9D4F41F3A0941A480F5EC933?method=download&shareKey=b6112fdd0a2fb53f08ec1a3197cc4f6b)

* * *

* 任务被重新调度到消费者2进行处理

![a3a59d7c980a40679d0fab6a5609213d.png](https://note.youdao.com/yws/api/personal/file/8B0FB9AC67744186A921CDCE06DEC28F?method=download&shareKey=4e2e2cec860fe3e74a04ea148d60763c)

* * *

### 3-消息持久化(或者叫队列和消息的持久化)

* 上面的例子中确保了即使消费者下线,任务也不会丢失.但是MQ服务停止,我们的任务任然会丢失

* * *

* 队列持久化可以通过在声明队列的时候进行持久化队列,如下图(修改生产者代码中的队列声明部分,消费者一样需要修改)

```java
//持久化队列声明,参数 durable 为 true
channel.queueDeclare(QUEUE_NAME, true, false, false, null);
```

* * *

* 消息持久化只需要修改生产者的消息发送部分,如下图

```java
//持久化消息声明,修改了BasicProperties参数
channel.basicPublish("", QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes(StandardCharsets.UTF_8))
```
* * *
### 4-公平派遣

* MQ的默认消息调度是轮训的,就像上面的例子,第一个任务给A如果有第二个消息,任务就给B,第三个消息任务就又给A.这样的调度会造成消费者的任务堆积假如在处理复杂任务的情况下,这种情况下可以在**消费者**中的队列中设置每次只接受一个消息,只有处理完过后才接受第二个消息

* 在**消费者**中设置如下

```java
int prefetchCount = 1 ;
channel.basicQos（prefetchCount）;
```
* * *

* 消费者如下图:

![ac6a9802e0bb383eff09834442e343a2.png](https://note.youdao.com/yws/api/personal/file/399F59017F2840A2ADFB435BADC17ED0?method=download&shareKey=713754d486f9955b36d41163151ceece)

* * *

* 这样的话在任务过多加上任务复杂的情况下可能会造成队列里面任务堆积太多,所以要及时的添加消费者
