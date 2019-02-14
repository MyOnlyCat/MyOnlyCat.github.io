---
layout: post
title: RabbitMQ笔记-4
categories: RabbitMQ
tags:
  - RabbitMQ
abbrlink: 50cfc150
date: 2019-02-14 00:00:00
---

**这里记录官方的第3个Demo,这个Demo主要讲了交换器在MQ中的使用**
<!-- more -->

[TOC]

### To-do List

- [x] 文章概要
- [ ] 交换器的类型
    - [x] 不使用交换器
    - [x] fanout交换器
    - [ ] direct交换器
    - [ ] topic交换器
    - [ ] headers交换器

* * *

### 0X00 概要,以及记录完成情况

* 在前两个Demo中**生产者**和**消费者**是通过**默认的交换器**,也就是空字符串(channel.basicPublish()中的**exchange**参数为""),前面有解释过当**exchange**参数为空字符串的情况下,消息是通过routingKey指定的名称路由到队列（如果存在,是直接路由到队列）,并没有通过交换器,而**MQ中消息传递模型核心思想是生产者永远不会降任何消息直接发送到队列.实际上，生产者通常甚至不知道消息是否会被传递到任何队列。**

* * *

* 相反,**生产者只能向交换器发送消息**,而交换器处理起来也非常的简单.**一方面它接收来自生产者的消息,另一方面它将接收到的这些消息推送到队列**,交换器呢也需要知道它应该如何去处理它收到的消息它应该附加到特定队列吗？它应该附加到许多队列吗？或者它应该被丢弃,**交换器的规则是由交换器的类型去定义**

### 0X01 交换器的类型

#### 默认交换器,或者叫不使用交换器

* 生产者

```java
//消息通过routingKey指定的名称路由到队列（如果存在,是直接路由到队列）
channel.basicPublish("", QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));
```

* * *

* 消费者

```java
channel.queueDeclare(QUEUE_NAME, false, false, false, null);
```

* * *

* 总结: 在默认交换器的情况下,生产者是直接将消息推送在队列,但是官方不推荐这样写


#### fanout类型的交换器

* 下图展示了fanout交换器的示意图

![fanout示意图](https://note.youdao.com/yws/api/personal/file/B1FF644BF82841D5BC9C6DB77BCD0E10?method=download&shareKey=1da32879131f58a553b0bc7496033720)

* * *

* fanout交换器也叫扇形交换器,**它会将收到的消息广播给所有的和它绑定的队列**,在上图中P只关心消息发送到哪个交换器,由交换器X去觉得把消息放到哪个队列,二C3 C4只关心自己订阅了哪个队列

* * *

##### 生产者代码编写

* 下面写一个完整的fanout交换器例子- **生产者**

```java
package demo03;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

/**
 * 官方的第三个Demo 生产者
 * 主要介绍了以下功能:
 * 1.Fanout交换器的使用
 */
public class FanoutSend {

    /**
     * 定义交换器名称
     */
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) {

        //配置连接信息
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("admin");
        factory.setPassword("123456");

        //try-with-resources
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel();
        ) {

            //声明交换器类型
            channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

            //定义消息
            String msg = "你好";

            //通过通道把消息发送给交换器
            channel.basicPublish(EXCHANGE_NAME, "", null, msg.getBytes(StandardCharsets.UTF_8));

            System.out.println(" [X] Sent '" + msg + "'");

        } catch (TimeoutException | IOException e) {
            e.printStackTrace();
        }
    }

}

```

* * *

##### 消费者代码编写

* 下面写一个完整的fanout交换器例子- **消费者**

```java
package demo03;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

/**
 * 对应官方Demo03的客户端
 */
public class FanoutRecv {

    /**
     * 交换器名称
     */
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws IOException, TimeoutException {

        //连接信息
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("admin");
        factory.setPassword("123456");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        //通道声明交换器
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

        //通道声明一个不为持久的队,独占的(仅限此连接),自动删除的(服务器不在使用或者下线将此队列删除)队列
        String queueName = channel.queueDeclare().getQueue();

        //将队列绑定到交换器
        channel.queueBind(queueName, EXCHANGE_NAME, "");

        System.out.println(" [*] 正在等待消息,退出请按 CTRL+C");

        //消息确认,关闭自动确认

//        DeliverCallback xx = new DeliverCallback() {
//            @Override
//            public void handle(String s, Delivery delivery) throws IOException {
//                String msg = new String(delivery.getBody(), StandardCharsets.UTF_8);
//                System.out.println(" [X] 收到消息'" + msg + "',我的工作编号为 FanoutRecv");
//                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
//            }
//        };
//        channel.basicConsume(queueName, false, xx, consumerTag -> {});

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String msg = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [X] 收到消息'" + msg + ",我的工作编号为 FanoutRecv'");
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        channel.basicConsume(queueName, false, deliverCallback, consumerTag -> {});
    }

}

```

* * *

#### 代码中的一些API介绍(基于5.6.0 AP)

##### 生产者的代码

> **channel.exchangeDeclare()**

* 此方法被重载了8次....精力有限,有能力可以参考[RabbitMQ-exchangeDeclare方法API](https://rabbitmq.github.io/rabbitmq-java-client/api/current/com/rabbitmq/client/Channel.html#exchangeDeclare(java.lang.String,com.rabbitmq.client.BuiltinExchangeType))

> channel.exchangeDeclare(java.lang.String exchange, java.lang.String type)

> 主动声明一个非自动删除，非持久的交换，没有额外的参数

| 参数名    |   参数类型    |   参数意义     |
| :-----:    |   :--------:   |   :--------:   |
| exchange  |   String      |   交换器的名称(比如上面生产者代码中的"logs")      |
| type      |   String      |   定义交换器的类型(代码中的"fanout"类型交换器)    |

* * * 

> **channel.basicPublish()**

* 此方法被重载了3次,参考[RabbitMQ-basicPublish方法API](https://rabbitmq.github.io/rabbitmq-java-client/api/current/com/rabbitmq/client/impl/recovery/AutorecoveringChannel.html#basicPublish(java.lang.String,java.lang.String,com.rabbitmq.client.AMQP.BasicProperties,byte[]))

> **channel.basicPublishjava.lang.String exchange, java.lang.String routingKey, AMQP.BasicProperties props, byte[] body)**

| 参数名    |   参数类型    |   参数意义     |
| :-----:    |   :--------:   |   :--------:   |
| exchange  |   String      |   消息需要发送到的交换器名称(比如上面生产者代码中的"logs")      |
| routingKek|   String      |   路由器密钥   |
| props     |   AMQP.BasicProperties      |   消息的其他属性 - 路由头等   |
| body      |   byte[]      |   消息正文   |

* * *

##### 消费者的代码

> **channel.basicAck()**

* 详细文档参考[RabbitMQ-basicAck方法API](https://rabbitmq.github.io/rabbitmq-java-client/api/current/com/rabbitmq/client/Channel.html#basicAck(long,boolean))

> **basicAck(long deliveryTag, boolean multiple)**

| 参数名    |   参数类型    |   参数意义     |
| :-----:    |   :--------:   |   :--------:   |
| deliveryTag  |   long      | RabbitMQ推送消息给消费者时，会附带一个DeliveryTag，以便消费者可以在消息确认时告诉 RabbitMQ 到底是哪条消息被确认了。RabbitMQ 保证在每个信道中，每条消息的 Delivery Tag 从 1 开始递增     |
| multiple|   boolean      |   取值为 false 时，表示通知 RabbitMQ 当前消息被确认；如果为 true，则额外将比第一个参数指定的 delivery tag 小的消息一并确认。（批量确认针对的是整个信道，参考gordon.study.rabbitmq.ack.TestBatchAckInOneChannel.java。） 对同一消息的重复确认，或者对不存在的消息的确认，会产生 IO 异常，导致信道关闭。  |

* * *

> **channel.basicConsume()**

* 此方法重载太多次没去数了...,详细文档参考[RabbitMQ-basicConsume方法API](https://rabbitmq.github.io/rabbitmq-java-client/api/current/com/rabbitmq/client/Channel.html#basicConsume(java.lang.String,boolean,com.rabbitmq.client.Consumer))

> **basicConsume(java.lang.String queue, boolean autoAck, DeliverCallback deliverCallback, CancelCallback cancelCallback)**

| 参数名    |   参数类型    |   参数意义     |
| :-----:   |   :--------:  |   :--------:   |
| queue     |   String      |   队列名称      |
| autoAck   |   boolean      |   是否自动确认消息(手动确认和自动确认,在开发中一般为手动确认,即值为false,详细说明参考[Consumer Acknowledgements and Publisher Confirms](https://www.rabbitmq.com/confirms.html))   |
| deliverCallback     |   deliverCallback      |   传递消息时的回调   |
| cancelCallback      |   CancelCallback      |    取消消费者时的回调(这个是个功能性接口可以用作lambda表达式或方法引用的赋值目标)   |

* * * 


