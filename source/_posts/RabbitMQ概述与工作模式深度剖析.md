---
title: RabbitMQ概述与工作模式深度剖析
toc: true
abbrlink: f73020b3
date: 2020-12-13 19:57:23
tags: RabbitMQ
categories: 消息中间件
---

#### 一、什么是消息中间件？
消息中间件是基于队列与消息传递技术，在网络环境中为应用系统提供同步或异步、可靠的消息传输的支撑性软件系统 。

优点：业务解耦、异步处理、削峰填谷
缺点：系统复杂度提高、系统可用性降低（消息丢失）
<!-- more -->
常见的MQ产品：
![image](/static/img/3.png)

#### 二、RabbitMQ简介
RabbitMQ是一个开源的消息代理和队列服务器，通过普通的协议(Amqp协议)来完成不同应用之间的数据共享（消费生产和消费者可以跨语言平台），RabbitMQ是通过elang语言来开发的基于AMQP协议。

AMQP，即 Advanced Message Queuing Protocol（高级消息队列协议），是一个网络协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。

RabbitMQ架构图：
![image](/static/img/5.png)

- Broker :接收和分发消息的应用，RabbitMQ Server就是 Message Broker
- Connection: 连接,应用程序与brokder建立网络连接
- Channel：网络通道，几乎所有的操作都是在channel中进行的，是进行消息对象的通道，客户端可以建立 多个通道，每一个channel表示一个会话任务
- Message: 服务器和应用程序之间传递数据的载体，有properties（消息属性,用来修饰消息,比如消息的优 先级,延时投递）和Body（消息体）
- Virtual host(虚拟主机): 出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个vhost，每个用户在自己的 vhost 创建 exchange／queue 等
- Exchange 交换机: 消息直接投递到交换机上，然后交换机根据消息的路由key 来路由到对应绑定的队列上
- Binding: 绑定exchange 与queue的虚拟连接,bingding中可以包含route_key
- route_key 路由key ，他的作用是在交换机上通过route_key来把消息路由到哪个队列上
- Queue：队列，用于来保存消息的载体，有消费者监听，然后消费消息

#### 三、RabbitMQ工作模式

https://www.rabbitmq.com/getstarted.html

##### 1、简单模式 Hello World
![image](/static/img/6.png)

在上图的模型中，有以下概念：
- P：生产者，也就是要发送消息的程序
- C：消费者：消息的接收者，会一直等待消息到来
- Queue：消息队列，图中红色部分。类似一个邮箱，可以缓存消息；生产者向其中投递消息，消费者从其中取出消息


代码示例：
```
<!--添加依赖-->
<dependency>
	<groupId>com.rabbitmq</groupId>
	<artifactId>amqp-client</artifactId>
   <version>5.7.1</version>
</dependency>

//生产者
package com.example.rabbitmqdemo.helloworld;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.ConnectionFactory;

public class Producer {

    private static String HELLO_QUEUE = "hello_queue";
    private static ConnectionFactory factory = new ConnectionFactory();

    static {
        factory.setHost("192.168.84.39");
        factory.setPort(5672);
        factory.setUsername("zhaojie");
        factory.setPassword("zhaojie");
        factory.setVirtualHost("/zhaojie");
    }

    public static void main(String[] args) throws Exception {
        // 简单模式
        // 生产者发送消息
        Channel channel = factory.newConnection().createChannel();
        //创建队列,声明并创建一个队列，如果队列已存在，则使用这个队列
        //第一个参数：队列名称ID
        //第二个参数：是否持久化，false对应不持久化数据，MQ停掉数据就会丢失
        //第三个参数：是否队列私有化，false则代表所有消费者都可以访问，true代表只有第一次拥有它的消费者才能一直使用，其他消费者不让访问
        //第四个参数：是否自动删除,false代表连接停掉后不自动删除掉这个队列
        //第五个参数：其他额外的参数, null
        channel.queueDeclare(HELLO_QUEUE, false, false, false, null);

        for (int i = 0; i < 100000; i++) {
            String message = "测试内容" + i;
            //发布消息
            //第一个参数：exchange 交换机，暂时用不到，在后面进行发布订阅时才会用到
            //第二个参数：队列名称
            //第三个参数：额外的设置属性
            //最后一个参数：要传递的消息字节数组
            channel.basicPublish("", HELLO_QUEUE, null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}


//消费者
package com.example.rabbitmqdemo.helloworld;

import com.rabbitmq.client.*;

import java.io.IOException;

public class Consumer {

    private static String HELLO_QUEUE = "hello_queue";
    private static ConnectionFactory factory = new ConnectionFactory();

    static {
        factory.setHost("192.168.84.39");
        factory.setPort(5672);
        factory.setUsername("zhaojie");
        factory.setPassword("zhaojie");
        factory.setVirtualHost("/zhaojie");
    }

    public static void main(String[] args) throws Exception {
        // 简单模式
        // 消费者消费消息
        Channel channel = factory.newConnection().createChannel();

        //创建队列,声明并创建一个队列，如果队列已存在，则使用这个队列
        //第一个参数：队列名称ID
        //第二个参数：是否持久化，false对应不持久化数据，MQ停掉数据就会丢失
        //第三个参数：是否队列私有化，false则代表所有消费者都可以访问，true代表只有第一次拥有它的消费者才能一直使用，其他消费者不让访问
        //第四个：是否自动删除,false代表连接停掉后不自动删除掉这个队列
        //其他额外的参数, null
        channel.queueDeclare(HELLO_QUEUE,false, false, false, null);

        //从MQ服务器中获取数据
        //创建一个消息消费者
        //第一个参数：队列名
        //第二个参数代表是否自动确认收到消息，false代表手动编程来确认消息，这是MQ的推荐做法
        //第三个参数要传入DefaultConsumer的实现类
        channel.basicConsume(HELLO_QUEUE, false, new Reciver(channel));
    }
}


class  Reciver extends DefaultConsumer {
    private Channel channel;
    //重写构造函数,Channel通道对象需要从外层传入，在handleDelivery中要用到
    public Reciver(Channel channel) {
        super(channel);
        this.channel = channel;
    }

    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        String message = new String(body);
        System.out.println("消费者接收到的消息："+message);
        System.out.println("消息的TagId："+envelope.getDeliveryTag());
        //false只确认签收当前的消息，设置为true的时候则代表签收该消费者所有未签收的消息
        channel.basicAck(envelope.getDeliveryTag(), false);
    }
}
```

##### 2、工作队列 Work queues
![image](/static/img/7.png)

- Work Queues：与入门程序的简单模式相比，多了一个或一些消费端，多个消费端共同消费同一个队列中的消息。
- 应用场景：对于任务过重或任务较多情况使用工作队列可以提高任务处理的速度。

注：每一条消息只能被一个消费者消费一次。

代码示例：
```
//生产者
package com.example.rabbitmqdemo.workqueues;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.ConnectionFactory;

public class Producer {

    private static String WORK_QUEUE = "work_queue";
    private static ConnectionFactory factory = new ConnectionFactory();

    static {
        factory.setHost("192.168.84.39");
        factory.setPort(5672);
        factory.setUsername("zhaojie");
        factory.setPassword("zhaojie");
        factory.setVirtualHost("/zhaojie");
    }

    public static void main(String[] args) throws Exception {
        // 工作队列模式
        // 生产者发送消息
        Channel channel = factory.newConnection().createChannel();
        channel.queueDeclare(WORK_QUEUE, false, false, false, null);

        for (int i = 0; i < 100000; i++) {
            String message = "测试内容" + i;
            channel.basicPublish("", WORK_QUEUE, null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}

//消费者1
package com.example.rabbitmqdemo.workqueues;

import com.rabbitmq.client.*;

import java.io.IOException;

public class Consumer1 {

    private static String WORK_QUEUE = "work_queue";
    private static ConnectionFactory factory = new ConnectionFactory();

    static {
        factory.setHost("192.168.84.39");
        factory.setPort(5672);
        factory.setUsername("zhaojie");
        factory.setPassword("zhaojie");
        factory.setVirtualHost("/zhaojie");
    }

    public static void main(String[] args) throws Exception {
        // 工作队列模式
        // 消费者1消费消息
        Channel channel = factory.newConnection().createChannel();
        channel.queueDeclare(WORK_QUEUE, false, false, false, null);

        //如果不写basicQos（1），则自动MQ会将所有请求平均发送给所有消费者
        //basicQos,MQ不再对消费者一次发送多个请求，而是消费者处理完一个消息后（确认后），在从队列中获取一个新的
        channel.basicQos(1);//处理完一个取一个
        channel.basicConsume(WORK_QUEUE, false, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String jsonSMS = new String(body);
                System.out.println("消费者1消费成功:" + jsonSMS);
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        });
    }
}

//消费者2
package com.example.rabbitmqdemo.workqueues;

import com.rabbitmq.client.*;

import java.io.IOException;

public class Consumer2 {

    private static String WORK_QUEUE = "work_queue";
    private static ConnectionFactory factory = new ConnectionFactory();

    static {
        factory.setHost("192.168.84.39");
        factory.setPort(5672);
        factory.setUsername("zhaojie");
        factory.setPassword("zhaojie");
        factory.setVirtualHost("/zhaojie");
    }

    public static void main(String[] args) throws Exception {
        // 工作队列模式
        // 消费者2消费消息
        Channel channel = factory.newConnection().createChannel();
        channel.queueDeclare(WORK_QUEUE, false, false, false, null);

        //如果不写basicQos（1），则自动MQ会将所有请求平均发送给所有消费者
        //basicQos,MQ不再对消费者一次发送多个请求，而是消费者处理完一个消息后（确认后），在从队列中获取一个新的
        channel.basicQos(1);//处理完一个取一个
        channel.basicConsume(WORK_QUEUE, false, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String jsonSMS = new String(body);
                System.out.println("消费者2消费成功:" + jsonSMS);
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        });
    }
}
```

##### 3、发布订阅 Publish/Subscribe
![image](/static/img/8.png)

在订阅模型中，多了一个 Exchange 角色，而且过程略有变化：
- P：生产者，也就是要发送消息的程序，但是不再发送到队列中，而是发给X（交换机）
- C：消费者，消息的接收者，会一直等待消息到来
- Queue：消息队列，接收消息、缓存消息
- Exchange：交换机（X）。一方面，接收生产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。Exchange有常见以下3种类型：
	- Fanout：广播，将消息交给所有绑定到交换机的队列
	- Direct：定向，把消息交给符合指定routing key 的队列
	- Topic：通配符，把消息交给符合routing pattern（路由模式） 的队列

Exchange（交换机）只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与 Exchange 绑定，或者没有符合路由规则的队列，那么消息会丢失！

代码示例：
```
//发布者
package com.example.rabbitmqdemo.publishsubscribe;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.ConnectionFactory;

public class Publisher {

    private static String PUBSUB_EXCHANGE = "pubsub_exchange";
    private static ConnectionFactory factory = new ConnectionFactory();

    static {
        factory.setHost("192.168.84.39");
        factory.setPort(5672);
        factory.setUsername("zhaojie");
        factory.setPassword("zhaojie");
        factory.setVirtualHost("/zhaojie");
    }

    public static void main(String[] args) throws Exception {
        // 发布订阅模式
        // 发布者发布消息 >> 交换机需要手动创建
        Channel channel = factory.newConnection().createChannel();
        for (int i = 0; i < 100; i++) {
            String message = "测试内容" + i;
            channel.basicPublish(PUBSUB_EXCHANGE, "", null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}

//订阅者1
package com.example.rabbitmqdemo.publishsubscribe;

import com.rabbitmq.client.*;

import java.io.IOException;

public class Subscribe1 {

    private static String PUBSUB_EXCHANGE = "pubsub_exchange";
    private static String SUBSCRIBE1_QUEUE = "subscribe1_queue";

    private static ConnectionFactory factory = new ConnectionFactory();

    static {
        factory.setHost("192.168.84.39");
        factory.setPort(5672);
        factory.setUsername("zhaojie");
        factory.setPassword("zhaojie");
        factory.setVirtualHost("/zhaojie");
    }

    public static void main(String[] args) throws Exception {
        // 发布订阅模式
        // 订阅者1消费消息
        Channel channel = factory.newConnection().createChannel();
        //声明队列信息
        channel.queueDeclare(SUBSCRIBE1_QUEUE, false, false, false, null);
        //queueBind用于将队列与交换机绑定
        //参数1：队列名 参数2：交互机名  参数三：路由key（暂时用不到)
        channel.queueBind(SUBSCRIBE1_QUEUE, PUBSUB_EXCHANGE, "");
        channel.basicQos(1);
        channel.basicConsume(SUBSCRIBE1_QUEUE, false, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("订阅者1消费消息：" + new String(body));
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        });
    }
}

//订阅者2
package com.example.rabbitmqdemo.publishsubscribe;

import com.rabbitmq.client.*;

import java.io.IOException;

public class Subscribe2 {

    private static String PUBSUB_EXCHANGE = "pubsub_exchange";
    private static String SUBSCRIBE2_QUEUE = "subscribe2_queue";

    private static ConnectionFactory factory = new ConnectionFactory();

    static {
        factory.setHost("192.168.84.39");
        factory.setPort(5672);
        factory.setUsername("zhaojie");
        factory.setPassword("zhaojie");
        factory.setVirtualHost("/zhaojie");
    }

    public static void main(String[] args) throws Exception {
        // 发布订阅模式
        // 订阅者2消费消息
        Channel channel = factory.newConnection().createChannel();
        //声明队列信息
        channel.queueDeclare(SUBSCRIBE2_QUEUE, false, false, false, null);
        //queueBind用于将队列与交换机绑定
        //参数1：队列名 参数2：交互机名  参数三：路由key（暂时用不到)
        channel.queueBind(SUBSCRIBE2_QUEUE, PUBSUB_EXCHANGE, "");
        channel.basicQos(1);
        channel.basicConsume(SUBSCRIBE2_QUEUE, false, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("订阅者2消费消息：" + new String(body));
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        });
    }
}
```

##### 4、路由 Routing
![image](/static/img/9.png)

##### 5、主题 Topics
![image](/static/img/10.png)

##### 6、远程调用 RPC（不常用）
![image](/static/img/11.png)
