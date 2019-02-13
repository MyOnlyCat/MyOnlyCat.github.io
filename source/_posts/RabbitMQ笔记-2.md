---
layout: post
title: RabbitMQ笔记-2
categories: RabbitMQ
tags:
  - RabbitMQ
abbrlink: b9ac6465
date: 2019-01-21 00:00:00
---

**这篇笔记主要记录MQ官方的第一个Demo,这里我用的是mvn构建**
<!--more-->

[TOC]

## demo-01-"Hello World!"

>在本教程的这一部分中，我们将用Java编写两个程序; 发送单个消息的生产者，以及接收消息并将其打印出来的消费者。我们将掩盖Java API中的一些细节，专注于这个非常简单的事情，只是为了开始。这是消息传递的“Hello World”。


* * *

* 官方的图片示例如下:

![Image](https://note.youdao.com/yws/api/personal/file/83019E106F814D8089E3145FD71B43B7?method=download&shareKey=0842d5eac632ebd4047592520ee8dd14)

* 在上图中,"P"为**生产者**,"C"为**消费者**.中间为**队列** -  RabbitMQ代表消费者保留的消息缓冲区。

### 1.pom引入jar

* 引入pom

```xml
<dependency>   
    <groupId>com.rabbitmq</groupId>  
    <artifactId>amqp-client</artifactId>    
    <version>5.5.3</version>
</dependency>

<dependency>    
    <groupId>org.slf4j</groupId>    
    <artifactId>slf4j-api</artifactId>    
    <version>1.7.25</version>
</dependency>

<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.25</version>
    <scope>test</scope>
</dependency>

<dependency>    
    <groupId>org.slf4j</groupId>   
    <artifactId>slf4j-nop</artifactId>   
    <version>1.7.22</version>
</dependency>



```

* * *

### 2.消息发送流程

* 图片示例如下

![消息发送](https://note.youdao.com/yws/api/personal/file/9877A3CD04384B9B9AB6468E11C7811A?method=download&shareKey=92932a78fa0f0ba6d62ef56b69ed0a1c)

* 上图中"P"为生产者正在向队列中存入消息,这里的java就要写出这个流程


#### 2.1-java生产者代码编写

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Send {
    
    /**
    ** 声明队列的名字
    **/
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) {

        //创建一个连接
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("demoUser");
        factory.setPassword("123456");

        //声明队列(声明队列是幂等性的,只有在它不存在的情况下才会创建它),注意这里使用了 try-with-resources,
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message = "Hello World!";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + message + "'");
        } catch (TimeoutException | IOException e) {
            e.printStackTrace();
        }

    }

}

```

### 3.消息接收流程

* **消费者**监听RabbitMQ的消息,因此与发布单个消息的发布者不同，我们将保持其运行以侦听消息并将其打印出来。

* 示例图如下

![消息监听](https://note.youdao.com/yws/api/personal/file/EAC7668BAC5E4F0F9165E38FD498D81C?method=download&shareKey=8b7786eaa0d6cad7b39df29d857dc96b)


#### 3.1-java消费者代码编写

```java

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Recv {

    /**
     * 队列名称,保持与send端一样
     */
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {

        //连接
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("demoUser");
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
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" + message + "'");
        };
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> {});

    }

}


```

### 4.运行效果

* **生产者**
![生产者](https://note.youdao.com/yws/api/personal/file/D7A933782B3C4DA2B6147D41B08A06A8?method=download&shareKey=82cb0c5360de0e7fe770b319759b7a39)

* * *

* **消费者**

![消费者](https://note.youdao.com/yws/api/personal/file/9CC6812A74E042FDA6B0C64695936502?method=download&shareKey=8ff7f6c47716b4f9f33bc32e5277716c)
