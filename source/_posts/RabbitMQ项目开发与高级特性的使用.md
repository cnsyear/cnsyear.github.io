---
title: RabbitMQ项目开发与高级特性的使用
toc: true
abbrlink: b8b454f2
date: 2020-12-15 15:37:47
tags: RabbitMQ
categories: 消息中间件
---

#### 一、Spring Boot2.x整合RabbitMQ

上文使用amqp-client演示[RabbitMQ工作模式](https://cnsyear.gitee.io/posts/f73020b3.html#%E4%B8%89%E3%80%81RabbitMQ%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F)的几种使用方式，下面将使用Spring Boot整合使用！！

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

#### 二、
