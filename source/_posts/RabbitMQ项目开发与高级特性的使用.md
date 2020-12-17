---
title: RabbitMQ项目开发与高级特性的使用
toc: true
tags: RabbitMQ
categories: 消息中间件
abbrlink: 3de380d5
date: 2020-12-16 14:11:31
---

#### 一、Spring Boot2.x整合RabbitMQ

上文使用amqp-client演示[RabbitMQ工作模式](https://cnsyear.gitee.io/posts/f73020b3.html#%E4%B8%89%E3%80%81RabbitMQ%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F)的几种使用方式和[RabbitMQ消息确认机制](https://cnsyear.gitee.io/posts/f73020b3.html#%E5%9B%9B%E3%80%81RabbitMQ%E6%B6%88%E6%81%AF%E7%A1%AE%E8%AE%A4%E6%9C%BA%E5%88%B6)，下面将讲解如何和Spring Boot整合使用！！

<!--more-->
```
//添加依赖
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>

//配置文件
spring:
  rabbitmq:
    host: 192.168.84.39
    port: 5672
    username: zhaojie
    password: zhaojie
    virtual-host: /zhaojie
```
#### 二、RabbitMQ工作模式

##### 1、简单模式 Hello World

```
//配置
package com.example.rabbitmqspringboot.helloworld;

import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HelloConfig {

    public static final String BOOT_HELLO_QUEUE = "BOOT_HELLO_QUEUE";

    @Bean(BOOT_HELLO_QUEUE)
    public Queue setQueue() {
        return new Queue(BOOT_HELLO_QUEUE);
    }
}

//生产者
package com.example.rabbitmqspringboot.helloworld;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;

@RestController
public class HelloProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/hello")
    public String hello() {
        String message = "hello "+ LocalDateTime.now();
        //设置部分请求参数
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN);
        //发消息
        rabbitTemplate.send(HelloConfig.BOOT_HELLO_QUEUE, new Message(message.getBytes(), messageProperties));
        return "message sended : " + message;
    }
}

//消费者
package com.example.rabbitmqspringboot.helloworld;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class HelloConcumer {

    //直连模式的多个消费者，会分到其中一个消费者进行消费。类似task模式
    //通过注入RabbitContainerFactory对象，来设置一些属性，相当于task里的channel.basicQos
    @RabbitListener(queues = HelloConfig.BOOT_HELLO_QUEUE)
    public void helloConsumer(String message) {
        System.out.println("hello received message : " + message);
    }
}
```

##### 2、工作队列 Work Queues

```
//配置
package com.example.rabbitmqspringboot.workqueue;

import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class WorkConfig {

    public static final String BOOT_WORK_QUEUE = "BOOT_WORK_QUEUE";

    @Bean(BOOT_WORK_QUEUE)
    public Queue setQueue() {
        return new Queue(BOOT_WORK_QUEUE);
    }
}

//生产者
package com.example.rabbitmqspringboot.workqueue;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;

@RestController
public class WorkProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/work")
    public String work() {
        String message = "work "+ LocalDateTime.now();
        //设置部分请求参数
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN);
        //发消息
        rabbitTemplate.send(WorkConfig.BOOT_WORK_QUEUE, new Message(message.getBytes(), messageProperties));
        return "message sended : " + message;
    }
}

//消费者
package com.example.rabbitmqspringboot.workqueue;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class WorkConcumer {

    //工作队列模式 消息会被平均分配到每一个消费者上
    @RabbitListener(queues = WorkConfig.BOOT_WORK_QUEUE)
    public void workConsumer1(String message) {
        System.out.println("work1 received message : " + message);
    }

    @RabbitListener(queues = WorkConfig.BOOT_WORK_QUEUE)
    public void workConsumer2(String message) {
        System.out.println("work2 received message : " + message);
    }
}
```

##### 3、发布订阅 Publish/Subscribe

```
//配置
package com.example.rabbitmqspringboot.publishsubscribe;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class PubSubConfig {

    public static final String BOOT_PUBSUB_QUEUE_1 = "BOOT_PUBSUB_QUEUE_1";
    public static final String BOOT_PUBSUB_QUEUE_2 = "BOOT_PUBSUB_QUEUE_2";
    public static final String BOOT_PUBSUB_EXCHANGE = "BOOT_PUBSUB_EXCHANGE";

    @Bean(BOOT_PUBSUB_QUEUE_1)
    public Queue fanoutQ1() {
        return new Queue(BOOT_PUBSUB_QUEUE_1);
    }

    @Bean(BOOT_PUBSUB_QUEUE_2)
    public Queue fanoutQ2() {
        return new Queue(BOOT_PUBSUB_QUEUE_2);
    }

    //声明exchange
    //Fanout模式需要声明exchange，并绑定queue，由exchange负责转发到queue上。
    //广播模式 交换机类型设置为：fanout
    @Bean(BOOT_PUBSUB_EXCHANGE)
    public FanoutExchange setFanoutExchange() {
        return new FanoutExchange(BOOT_PUBSUB_EXCHANGE);
    }

    //声明Binding,exchange与queue的绑定关系
    @Bean
    public Binding bindFanoutQ1() {
        return BindingBuilder.bind(fanoutQ1()).to(setFanoutExchange());
    }

    @Bean
    public Binding bindFanoutQ2() {
        return BindingBuilder.bind(fanoutQ2()).to(setFanoutExchange());
    }
}

//生产者
package com.example.rabbitmqspringboot.publishsubscribe;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;

@RestController
public class PubSubProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/pubsub")
    public String pubsub() {
        String message = "pubsub " + LocalDateTime.now();
        //设置部分请求参数
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN);
        //发消息
        rabbitTemplate.send(PubSubConfig.BOOT_PUBSUB_EXCHANGE, "", new Message(message.getBytes(), messageProperties));
        return "message sended : " + message;
    }
}

//消费者
package com.example.rabbitmqspringboot.publishsubscribe;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class PubSubConcumer {

    @RabbitListener(queues = PubSubConfig.BOOT_PUBSUB_QUEUE_1)
    public void pubsub1Consumer(String message) {
        System.out.println("pubsub1 received message : " + message);
    }

    @RabbitListener(queues = PubSubConfig.BOOT_PUBSUB_QUEUE_2)
    public void pubsub2Consumer(String message) {
        System.out.println("pubsub2 received message : " + message);
    }
}
```

##### 4、路由 Routing

```
//配置
package com.example.rabbitmqspringboot.routing;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RoutingConfig {

    public static final String BOOT_DIRECT_QUEUE_1 = "BOOT_DIRECT_QUEUE_1";
    public static final String BOOT_DIRECT_QUEUE_2 = "BOOT_DIRECT_QUEUE_2";
    public static final String BOOT_DIRECT_EXCHANGE = "BOOT_DIRECT_EXCHANGE";

    @Bean(BOOT_DIRECT_QUEUE_1)
    public Queue directQ1() {
        return new Queue(BOOT_DIRECT_QUEUE_1);
    }

    @Bean(BOOT_DIRECT_QUEUE_2)
    public Queue directQ2() {
        return new Queue(BOOT_DIRECT_QUEUE_2);
    }

    //声明exchange
    //Direct模式需要声明exchange，并绑定queue，由exchange负责转发到queue上。
    //路由模式 交换机类型设置为：direct
    @Bean(BOOT_DIRECT_EXCHANGE)
    public DirectExchange setDirectExchange() {
        return new DirectExchange(BOOT_DIRECT_EXCHANGE);
    }

    //声明Binding,exchange与queue的绑定关系并添加路由key
    @Bean
    public Binding bindDirectQ1() {
        return BindingBuilder.bind(directQ1()).to(setDirectExchange()).with("direct.key1");
    }

    @Bean
    public Binding bindDirectQ2() {
        return BindingBuilder.bind(directQ2()).to(setDirectExchange()).with("direct.key2");
    }
}

//生产者
package com.example.rabbitmqspringboot.routing;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RoutingProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/routing")
    public String routing() {
        //设置部分请求参数
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN);
        //发消息
        rabbitTemplate.send(RoutingConfig.BOOT_DIRECT_EXCHANGE, "direct.key1", new Message("key1消息".getBytes(), messageProperties));
        rabbitTemplate.send(RoutingConfig.BOOT_DIRECT_EXCHANGE, "direct.key2", new Message("key2消息".getBytes(), messageProperties));
        return "message sended";
    }
}

//消费者
package com.example.rabbitmqspringboot.routing;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class RoutingConcumer {
    //Routing路由模式
    @RabbitListener(queues = RoutingConfig.BOOT_DIRECT_QUEUE_1)
    public void pubsub1Consumer(String message) {
        System.out.println("routing1 只接收direct.key1 received message : " + message);
    }

    @RabbitListener(queues = RoutingConfig.BOOT_DIRECT_QUEUE_2)
    public void pubsub2Consumer(String message) {
        System.out.println("routing2 只接收direct.key2 received message : " + message);
    }
}
```

##### 5、主题 Topics

```
//配置
package com.example.rabbitmqspringboot.topic;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class TopicConfig {

    public static final String BOOT_TOPIC_QUEUE_1 = "BOOT_TOPIC_QUEUE_1";
    public static final String BOOT_TOPIC_QUEUE_2 = "BOOT_TOPIC_QUEUE_2";
    public static final String BOOT_TOPIC_EXCHANGE = "BOOT_TOPIC_EXCHANGE";

    @Bean(BOOT_TOPIC_QUEUE_1)
    public Queue topicQ1() {
        return new Queue(BOOT_TOPIC_QUEUE_1);
    }

    @Bean(BOOT_TOPIC_QUEUE_2)
    public Queue topicQ2() {
        return new Queue(BOOT_TOPIC_QUEUE_2);
    }

    //Topics模式 交换机类型设置为：topic
    @Bean(BOOT_TOPIC_EXCHANGE)
    public TopicExchange setTopicExchange() {
        return new TopicExchange(BOOT_TOPIC_EXCHANGE);
    }

    //声明Binding,exchange与queue的绑定关系并添加路由key规则
    @Bean
    public Binding bindTopicQ1() {
        return BindingBuilder.bind(topicQ1()).to(setTopicExchange()).with("topic.#");
    }

    @Bean
    public Binding bindTopicQ2() {
        return BindingBuilder.bind(topicQ2()).to(setTopicExchange()).with("topic.*");
    }
}

//生产者
package com.example.rabbitmqspringboot.topic;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TopicProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/topic")
    public String topic() {
        //设置部分请求参数
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN);
        //发消息
        rabbitTemplate.send(TopicConfig.BOOT_TOPIC_EXCHANGE, "topic.key1", new Message("key1消息".getBytes(), messageProperties));
        rabbitTemplate.send(TopicConfig.BOOT_TOPIC_EXCHANGE, "topic.key2", new Message("key2消息".getBytes(), messageProperties));
        rabbitTemplate.send(TopicConfig.BOOT_TOPIC_EXCHANGE, "topic.key2.a", new Message("key2消息a".getBytes(), messageProperties));
        rabbitTemplate.send(TopicConfig.BOOT_TOPIC_EXCHANGE, "topic.key2.b", new Message("key2消息b".getBytes(), messageProperties));
        rabbitTemplate.send(TopicConfig.BOOT_TOPIC_EXCHANGE, "topic.key2.c", new Message("key2消息c".getBytes(), messageProperties));
        return "message sended";
    }
}

//消费者
package com.example.rabbitmqspringboot.topic;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class TopicConcumer {
    //Topic主题模式
    //注意这个模式会有优先匹配原则。例如发送routingKey=hunan.IT,那匹配到hunan.*(hunan.IT,hunan.eco),之后就不会再去匹配*.ITd
    @RabbitListener(queues = TopicConfig.BOOT_TOPIC_QUEUE_1)
    public void pubsub1Consumer(String message) {
        System.out.println("topic1 只接收topic.# 匹配一个单词 也可以匹配多个单词  received message : " + message);
    }

    @RabbitListener(queues = TopicConfig.BOOT_TOPIC_QUEUE_2)
    public void pubsub2Consumer(String message) {
        System.out.println("topic2 只接收topic.* 匹配一个单词 received message : " + message);
    }
}

//运行结果
topic1 只接收topic.# 匹配一个单词 也可以匹配多个单词  received message : key1消息
topic2 只接收topic.* 匹配一个单词 received message : key1消息
topic1 只接收topic.# 匹配一个单词 也可以匹配多个单词  received message : key2消息
topic2 只接收topic.* 匹配一个单词 received message : key2消息
topic1 只接收topic.# 匹配一个单词 也可以匹配多个单词  received message : key2消息a
topic1 只接收topic.# 匹配一个单词 也可以匹配多个单词  received message : key2消息b
topic1 只接收topic.# 匹配一个单词 也可以匹配多个单词  received message : key2消息c
```

#### 三、RabbitMQ消息确认机制

该示例基于4、路由 Routing例子

##### 1、生产者Confirm和Return机制

在使用 RabbitMQ 的时候，作为消息发送方希望杜绝任何消息丢失或者投递失败场景。RabbitMQ 为我们提供了Confirm和Return机制用来控制消息的投递可靠性模式。

RabbitMQ整个消息投递的路径为：producer--->RabbitMQ broker--->exchange--->queue--->consumer

- 消息从 producer 到 exchange 则会返回一个 confirmCallback
- 消息从 exchange-->queue 投递失败则会返回一个 returnCallback

我们将利用这两个 callback 控制消息的可靠性投递

- 使用rabbitTemplate.setConfirmCallback设置回调函数。当消息发送到exchange后回调confirm方法。在方法中判断ack，如果为true，则发送成功，如果为false，则发送失败，需要处理。
- 使用rabbitTemplate.setReturnCallback设置退回函数，当消息从exchange路由到queue失败后，如果设置了rabbitTemplate.setMandatory(true)参数，则会将消息退回给producer。并执行回调函数returnedMessage

```
//配置文件
spring:
  rabbitmq:
    host: 192.168.84.39
    port: 5672
    username: zhaojie
    password: zhaojie
    virtual-host: /zhaojie
    #发布者确认
    publisher-confirm-type: correlated
    #发布者到达确认
    publisher-returns: true

//生产者回调方法
package com.example.rabbitmqspringboot.routing;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Component;

/**
 * 生产者确认-confirm和return机制
 */
@Component
public class PublisherCallback implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnCallback {

    private Logger log = LoggerFactory.getLogger(PublisherCallback.class);

    /**
     * confirm机制只保证消息到达exchange，不保证消息可以路由到正确的queue,如果exchange错误，就会触发confirm机制
     *
     * @param correlationData
     * @param ack
     * @param cause
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if (!ack) {
            log.error("rabbitmq confirm fail,cause:{}", cause);
        }else {
            log.info("rabbitmq confirm success,cause:{}", cause);
        }
    }
    
    /**
     * Return 消息机制用于处理一个不可路由的消息。在某些情况下，如果我们在发送消息的时候，当前的 exchange 不存在或者指定路由 key 路由不到，
     * 这个时候我们需要监听这种不可达的消息就需要这种return机制
     *
     * @param message
     * @param replyCode
     * @param replyText
     * @param exchange
     * @param routingKey
     */
    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
        log.error("mq消息不可达,message:{},replyCode:{},replyText:{},exchange:{},routing:{}", message.toString(), replyCode, replyText, exchange, routingKey);
        String messageId = message.getMessageProperties().getMessageId();
    }
}

//RabbitTemplate配置
package com.example.rabbitmqspringboot.routing;

import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitTemplateConfig {

    /**
     * 设置返回回调和确认回调
     *
     * @param connectionFactory
     * @return
     */
    @Bean
    RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory, PublisherCallback publisherCallback) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate();
        rabbitTemplate.setConnectionFactory(connectionFactory);
        rabbitTemplate.setConfirmCallback(publisherCallback);
        rabbitTemplate.setReturnCallback(publisherCallback);
        //Mandatory为true时,消息通过交换器无法匹配到队列会返回给生产者，为false时匹配不到会直接被丢弃
        rabbitTemplate.setMandatory(true);
        return rabbitTemplate;
    }
}


//生产者
package com.example.rabbitmqspringboot.routing;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RoutingProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/routing")
    public String routing() {
        //设置部分请求参数
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN);
        //没有对应的交换机，不能正常发送 会触发Confirm机制
        //运行结果
        //Channel shutdown: channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no exchange 'BOOT_DIRECT_EXCHANGE11112312' in vhost '/zhaojie', class-id=60, method-id=40)
        //rabbitmq confirm fail,cause:channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no exchange 'BOOT_DIRECT_EXCHANGE11112312' in vhost '/zhaojie', class-id=60, method-id=40)
        rabbitTemplate.send(RoutingConfig.BOOT_DIRECT_EXCHANGE + "11112312", "direct.key", new Message("key2消息".getBytes(), messageProperties));

        //能发送给交换机但是交换机不存在指定的路由key，消息不可达消息会触发Return机制
        //运行结果
        //rabbitmq confirm success,cause:null
        //mq消息不可达,message:(Body:'key2消息' MessageProperties [headers={}, contentType=text/plain, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, deliveryTag=0]),replyCode:312,replyText:NO_ROUTE,exchange:BOOT_DIRECT_EXCHANGE,routing:direct.key2111111
        rabbitTemplate.send(RoutingConfig.BOOT_DIRECT_EXCHANGE, "direct.key2111111", new Message("key2消息".getBytes(), messageProperties));
        return "message sended";
    }
}
```

##### 2、消费者手动Ack模式
Ack指Acknowledge，确认。 表示消费端收到消息后的确认方式。
有三种确认方式：
- 自动确认：acknowledge="none"
- 手动确认：acknowledge="manual"
- 根据异常情况确认：acknowledge="auto"，（这种方式使用麻烦，不作讲解）

其中自动确认是指，当消息一旦被Consumer接收到，则自动确认收到，并将相应 message 从 RabbitMQ 的消息缓存中移除。但是在实际业务处理中，很可能消息接收到，业务处理出现异常，那么该消息就会丢失。如果设置了手动确认方式，则需要在业务处理成功后，调用channel.basicAck()，手动签收，如果出现异常，则调用channel.basicNack()方法，让其自动重新发送消息。

```
//配置文件
spring:
  rabbitmq:
    host: 192.168.84.39
    port: 5672
    username: zhaojie
    password: zhaojie
    virtual-host: /zhaojie
    #发布者确认
    publisher-confirm-type: correlated
    #发布者到达确认
    publisher-returns: true
    listener:
      simple:
        #simple模式 关闭自动ack,手动ack
        acknowledge-mode: manual
        retry:
          #开启重试机制
          enabled: true
          #最大重试传递次数
          max-attempts: 3
          #第一次和第二次尝试传递消息的间隔时间 单位毫秒
          initial-interval: 5000ms
          #最大重试时间间隔，单位毫秒
          max-interval: 10000ms
          #重试间隔的乘法器，multiplier默认为1 间隔 0s 5s 15s
          multiplier: 3
        #重试次数超过上面的设置之后是否丢弃(消费者listener抛出异常，是否重回队列，默认true：重回队列， false为不重回队列(结合死信交换机))
        default-requeue-rejected: true

//消费者
package com.example.rabbitmqspringboot.routing;

import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
public class RoutingConcumer {
    //Routing路由模式
    @RabbitListener(queues = RoutingConfig.BOOT_DIRECT_QUEUE_1)
    public void pubsub1Consumer(String msg, Channel channel, Message message) throws IOException {
        System.out.println("手动ACK-----routing1 只接收direct.key1 received message : " + msg);
        //手动ack
        channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
    }

    @RabbitListener(queues = RoutingConfig.BOOT_DIRECT_QUEUE_2)
    public void pubsub2Consumer(String msg, Channel channel, Message message) throws IOException {
        try {
            System.out.println("出现异常后不会执行手动ACK，MQ会被重复尝试投递-----routing2 只接收direct.key2 received message : " + msg);
            //出现异常，不会执行手动ack
            int a =1/0;
            //手动ack
            channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
        } catch (IOException e) {
            // 拒绝当前消息，并把消息返回原队列
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
        }
    }
}

//生产者
package com.example.rabbitmqspringboot.routing;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RoutingProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/routing")
    public String routing() {
        //设置部分请求参数
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN);
        //正常执行手动ACK
        rabbitTemplate.send(RoutingConfig.BOOT_DIRECT_EXCHANGE, "direct.key1", new Message("key1消息".getBytes(), messageProperties));
        //出现异常后不会执行手动ACK，MQ会被重复尝试投递
        rabbitTemplate.send(RoutingConfig.BOOT_DIRECT_EXCHANGE, "direct.key2", new Message("key2消息".getBytes(), messageProperties));
        return "message sended";
    }
}

//运行结果
//手动ACK-----routing1 只接收direct.key1 received message : key1消息
//出现异常后不会执行手动ACK，MQ会被重复尝试投递-----routing2 只接收direct.key2 received message : key2消息
//出现异常后不会执行手动ACK，MQ会被重复尝试投递-----routing2 只接收direct.key2 received message : key2消息
//出现异常后不会执行手动ACK，MQ会被重复尝试投递-----routing2 只接收direct.key2 received message : key2消息
```

可以看到该消息状态为unacked,由于配置了default-requeue-rejected: true不会被丢弃，每次重启后会再次尝试发送该条消息！！

这里的手动ack模式都是使用yml配置文件的方式指定的，如果你是使用代码的方式注入的，要注意代码优于配置，会覆盖配置文件。
正常处理流程都是如果消费端没有出现异常，则调用channel.basicAck(deliveryTag,false)方法确认签收消息; 如果消费端出现异常，则在catch中调用 basicNack或 basicReject让MQ重新发送消息。

![image](/static/img/14.png)

#### 四、RabbitMQ消费端限流
假设一个场景，首先，我们 Rabbitmq 服务器积压了有上万条未处理的消息，我们随便打开一个消费者客户端，会出现这样情况: 巨量的消息瞬间全部推送过来，但是我们单个客户端无法同时处理这么多数据!

当数据量特别大的时候，我们对生产端限流肯定是不科学的，因为有时候并发量就是特别大，有时候并发量又特别少，我们无法约束生产端，这是用户的行为。所以我们应该对消费端限流，用于保持消费端的稳定，当消息数量激增的时候很有可能造成资源耗尽，以及影响服务的性能，导致系统的卡顿甚至直接崩溃。

RabbitMQ提供了一种qos（服务质量保证）功能，即在非自动确认消息的前提下，如果一定数目的消息（通过基于consumer或者channel设置qos的值）未被确认前，不进行消费新的消息。

该示例基于2、工作队列 Work Queues

```
#注意：消费端的确认模式一定为手动确认。acknowledge="manual"，这里的设置模式是全局的会对所有的队列生效！！！注意一般需要单独设置！！
#消费者每次从队列获取的消息数量 (默认一次250个)
spring.rabbitmq.listener.simple.prefetch=10

//生产者发送100条消息测试消费者每次消费消息的数量
package com.example.rabbitmqspringboot.workqueue;

import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
public class WorkConcumer {

    //工作队列模式 消息会被平均分配到每一个消费者上
    @RabbitListener(queues = WorkConfig.BOOT_WORK_QUEUE)
    public void workConsumer1(String msg, Channel channel, Message message) throws IOException {
        //模拟测试
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
	//手动签收
	channel.basicAck(message.getMessageProperties().getDeliveryTag(), true);
        System.out.println("work1 received message : " + msg);
    }
}
```
我们从下图中发现 Unacked值一直都是10，Unacked的值在这里代表消费者正在处理的消息，消费者一次性最多处理10条消息，达到了消费者限流的预期功能。
![image](/static/img/15.png)

#### 五、RabbitMQ过期时间(TTL)
TTL 全称 Time To Live（存活时间/过期时间）。
当消息到达存活时间后，还没有被消费，会被自动清除。
RabbitMQ可以对消息设置过期时间，也可以对整个队列（Queue）设置过期时间。

该示例基于2、工作队列 Work Queues

```
//配置
package com.example.rabbitmqspringboot.workqueue;

import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class WorkConfig {

    public static final String BOOT_WORK_QUEUE = "BOOT_WORK_QUEUE";

    @Bean(BOOT_WORK_QUEUE)
    public Queue setQueue() {

        /*Map<String, Object> args = new HashMap<>(2);
        //声明过期时间5秒
        args.put("x-message-ttl", 5000);
        return QueueBuilder.durable(BOOT_WORK_QUEUE).withArguments(args).build();*/
        
        Queue queue = new Queue(BOOT_WORK_QUEUE);
        //给队列设置过期时间 单位毫秒
        queue.addArgument("x-message-ttl", 30 * 1000);
        return queue;
    }
}

// 测试消息过期需要注释掉消费端代码，否则看不出效果

```
如果BOOT_WORK_QUEUE 之前创建过需要删除，重新启动会发现新创建的队列会有 TTL标记，测试发送消息如果消息超过30秒未被消费，就会被丢弃。
- 设置队列过期时间使用参数：x-message-ttl，单位：ms(毫秒)，会对整个队列消息统一过期。
- 设置消息过期时间使用参数：expiration。单位：ms(毫秒)，当该消息在队列头部时（消费时），会单独判断这一消息是否过期。
如果两者都进行了设置，以时间短的为准。

![image](/static/img/16.png)





#### 六、RabbitMQ死信队列


#### 七、RabbitMQ延迟队列
