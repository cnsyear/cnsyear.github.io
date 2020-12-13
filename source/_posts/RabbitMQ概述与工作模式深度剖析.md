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
RabbitMQ是一个开源的消息代理和队列服务器，通过普通的协议(Amqp协议)来完成不同应用之间的数据共享（消费生产和消费者可以跨语言平台），RabbitMQ是通过elang语言来开发的基于amqp协议。

