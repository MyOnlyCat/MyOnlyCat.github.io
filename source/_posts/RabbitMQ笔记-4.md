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
    - [x] direct交换器
    - [x] topic交换器
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

#### direct类型的交换器

* 下图展示了direct交换器的原理图

![direct交换器原理图](https://note.youdao.com/yws/api/personal/file/7B0BE23A61D64CEBA89F2A6576507AC5?method=download&shareKey=c2719152c3ff847c07969ed574025e2e)

* * *

* 上图可以看到 `direct类型的交换器`x和两个通道绑定,分别是Q1和Q2,第一个队列Q1绑定`orange`的`路由key(routingKey)`,而Q2则绑定的`black`和`green`的`路由key`

* 在这种配置下`P(生产者)`发送消息到`X(direct交换器)`,交换器根据消息的`routingKey(路由key)`的不同将消息分配到**不同的队列(Q1,Q2)**

* 图中`orange`key的消息被分配到了`Q1`,因为Q1队列绑定的`orange`;而`black,green`被交换器分配到了`Q2`,因为Q2队列绑定了`black`和`green`的`路由key`

* **使用相同的`路由key`去绑定多个队列是完全没有问题的**,如下图所示

![相同路由key绑定多个队列](https://note.youdao.com/yws/api/personal/file/678239E3A2024657AC09DC161DE6264B?method=download&shareKey=a296e2215ac93735c82569376d7f41ac)

* 上图中`Q1,Q2`都绑定了路由key`black`,在这种情况下,`direct`交换器类型就表现的和`fanout`类型一样了

##### 生产者代码编写

* 下面为Driect类型的交换器的生产者代码

```java
package demo04;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

/**
 * 官方的第4个Demo
 * 主要实现了以下内容:
 * 1.direct类型交换器的使用
 */
public class DirectSend {

    /**
     * 定义交换器的名称
     */
    public static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) {

        //配置连接信息
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("admin");
        factory.setPassword("123456");

        //try-with-resources
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()
        ) {

            //声明交换器类型
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

            //定义Info消息(消费者只有在routingKek相同时才能接收到消息)
            String infoMsg = "你好 direct";

            //定义Error消息
            String errorMsg = "错误 direct";

            //消息发送
            channel.basicPublish(EXCHANGE_NAME, "info", null, infoMsg.getBytes(StandardCharsets.UTF_8));

            System.out.println(" [X] Sent '" + infoMsg + "'");

            //错误消息发送
            channel.basicPublish(EXCHANGE_NAME, "error", null, errorMsg.getBytes(StandardCharsets.UTF_8));

            System.out.println(" [X] Sent '" + errorMsg + "'");

        } catch (TimeoutException | IOException e) {
            e.printStackTrace();
        }

    }

}

```

##### 消费者代码编写

###### 负责接收Info路由key

```java
package demo04;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

/**
 * 对应官方Demo04的客户端
 * 负责接收Info
 */
public class DirectRecvForInfo {

    /**
     * 交换器名称
     */
    public static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws IOException, TimeoutException {

        //配置连接信息
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("admin");
        factory.setPassword("123456");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        //声明交换器
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

        //通过通道声明一个不为持久的队,独占的(仅限此连接),自动删除的(服务器不在使用或者下线将此队列删除)队列
        String queueName = channel.queueDeclare().getQueue();

        //绑定到交换器
        channel.queueBind(queueName, EXCHANGE_NAME, "info");

        System.out.println(" [*] 正在等待消息,退出请按 CTRL+C");

        DeliverCallback deliverCallback = (consumerTar, delivery) -> {
            String msg = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [X] 收到消息'" + msg + ",我的工作编号为 DirectRecv,负责接收 Info 消息'");
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        channel.basicConsume(queueName, false, deliverCallback, consumerTag -> {});
    }
}

```

* * * 

###### 负责接收Error路由key

```java
package demo04;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

/**
 * 对应官方Demo04的客户端
 * 负责接error
 */
public class DirectRecvForError {

    /**
     * 交换器名称
     */
    public static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws IOException, TimeoutException {

        //配置连接信息
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("admin");
        factory.setPassword("123456");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        //声明交换器
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

        //通过通道声明一个不为持久的队,独占的(仅限此连接),自动删除的(服务器不在使用或者下线将此队列删除)队列
        String queueName = channel.queueDeclare().getQueue();

        //绑定到交换器
        channel.queueBind(queueName, EXCHANGE_NAME, "error");

        System.out.println(" [*] 正在等待消息,退出请按 CTRL+C");

        DeliverCallback deliverCallback = (consumerTar, delivery) -> {
            String msg = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [X] 收到消息'" + msg + ",我的工作编号为 DirectRecv,负责接收 Error 消息'");
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        channel.basicConsume(queueName, false, deliverCallback, consumerTag -> {});
    }
}

```

##### 代码效果展示

* 生产者发送消息

![Direct类型交换器生产者发送消息](https://note.youdao.com/yws/api/personal/file/75DA5B3B31794D60B29C592FA40F3F05?method=download&shareKey=fda4f310bc855e80ec0a2d0cf67ea1ef)

* 消费者负责Info类型接收消息

![Direct类型交换器Info消费者接受消息](https://note.youdao.com/yws/api/personal/file/170B0FFCB34D4A1FBC1F671064301D7C?method=download&shareKey=5a901136cd9cea3ba4d95e11d8260cb1)

* 消费者负责Error类型接收消息

![Direct类型交换器消费者接受Error消息](https://note.youdao.com/yws/api/personal/file/461EA8FA6C23465A993B46A05EC771D6?method=download&shareKey=3bf4d348da0eedcc6db9f85915314856)

* **这里说一下上面介绍Direct交换器没有说到的地方就拿这个代码举例,当直接启动生产者发送消息,不启动两个消费者消息会丢失,只启动一个消费者,发送给另一个消费者的消息会丢失**

##### 代码中的一些API介绍(基于5.6.0 AP)

* 这里就暂时只说一下上面没说的,有空我会把API整合到一个地方

###### 消费者的代码

> **channel.queueBind()**

* 此方法被重载了2次....精力有限,有能力可以参考[RabbitMQ-queueBind方法API](https://rabbitmq.github.io/rabbitmq-java-client/api/current/com/rabbitmq/client/Channel.html#queueBind(java.lang.String,java.lang.String,java.lang.String))

> channel.queueBind(java.lang.String queue, java.lang.String exchange, java.lang.String routingKey)

> 主动声明一个非自动删除，非持久的交换，没有额外的参数

| 参数名    |   参数类型    |   参数意义     |
| :-----:    |   :--------:   |   :--------:   |
| queue  |   String      |   通道名称      |
| exchange      |   String      | 交换器名称    |
| routingKey      |   String      |   路由Key    |
* * * 

#### topic类型的交换器

* 下图展示了topic交换器的原理

![topic原理图](https://note.youdao.com/yws/api/personal/file/1037553C4D374527875FC64532594CD3?method=download&shareKey=a90964cdbe48eebb5becdbfd046e7147)

* 消息分发规则: **一个附带特殊的选择键将会被转发到绑定键与之匹配的队列中。**
* * *
* 路由key规则:**路由key必须是由点隔开的一系列的标识符组成：”stock.usd.nyse”,“nyse.vmw”,”quick.orange.rabbit”.你可以定义任数量的标识符，上限为255个字节。**
* * *
* 通道绑定的key: 这个有点类似于正则表达式的意思了,如下表:

| 表达式 | 匹配意义 |
| :--: | :--: |
| * | (星号)可以替代一个单词(或者标识符) |
| # | (hash)可以替换零个或多个单词(或者标识符) |

* * *

* 图示意义解释: **消息1(fast.orange.*)通过交换器(topic类型)分配到消费者Q6(#)和Q7(*.orange.*),因为Q6的#(hash)可以匹配任意标识符,而Q7的orange.(星号)匹配前后的一个单词或者标识符,所以这两个通道都能收到消息,消息2(lazy.orange.a.b)通过交换器会到达Q6.Q8,因为Q6匹配所有,而Q8(lazy.#)可以匹配lazy.后面的零个或者多个标识符**

##### 生产者代码编写

```java
package demo05;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class TopicSend {

    //定义交换器名称
    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) {

        //配置连接信息
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("admin");
        factory.setPassword("123456");

        //tyr-witch-resources
        try (Connection connection = factory.newConnection(); Channel channel = connection.createChannel();)
        {

            //声明交换器类型
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);

            //定义消息
            String msg = "# 匹配";

            //消息发送
            channel.basicPublish(EXCHANGE_NAME, "anonymous.info", null, msg.getBytes(StandardCharsets.UTF_8));

            System.out.println(" [X] Sent '" + msg + "'");

        } catch (TimeoutException | IOException e) {
            e.printStackTrace();
        }
    }

}

```

##### 消费者代码编写

```java
package demo05;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

public class TopicRecvForJinHao {

    /**
     * 交换器名称
     */
    public static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws IOException, TimeoutException {

        //配置连接信息
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("admin");
        factory.setPassword("123456");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        //声明交换器
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);

        //通过通道声明一个不为持久的队,独占的(仅限此连接),自动删除的(服务器不在使用或者下线将此队列删除)队列
        String queueName = channel.queueDeclare().getQueue();

        //绑定到交换器,配置匹配key为 # 接收所有类型消息
        channel.queueBind(queueName, EXCHANGE_NAME, "#");

        System.out.println(" [*] 正在等待消息,退出请按 CTRL+C");

        DeliverCallback deliverCallback = (consumerTar, delivery) -> {
            String msg = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [X] 收到消息'" + msg + ",我的路由Key为 # ,负责接收所有消息'");
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        channel.basicConsume(queueName, false, deliverCallback, consumerTag -> {});
    }

}

```
##### 代码效果展示

* 生产者发送消息如图所示
![topic发送消息](https://note.youdao.com/yws/api/personal/file/18262FE55C6941C1BEC22B5707A1D1B4?method=download&shareKey=2ec22d32cafbf26bfb06b2d7bba17a38)

* 消费者#号接收消息
![topic消费者#号匹配消息](https://note.youdao.com/yws/api/personal/file/12D9F659DE1C471986B8A44A97DC5942?method=download&shareKey=e1d8f21f3d0df6be88cc721a5896383b)

##### 其它说明

* 上面的代码只是一个最简单的demo状态而且只是实验了 # 匹配而已,你们可以启动两个生产者并且使用不同的路由key,然后在启动两个消费者使用不同的匹配规则,就可以实现类似于示例图的效果了