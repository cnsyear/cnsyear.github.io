---
title: CentOS7安装RabbitMQ3.6.5
toc: true
abbrlink: 64fadf5f
date: 2020-12-13 20:31:56
tags: RabbitMQ
categories: 
---

#### 下载相关文件
https://www.rabbitmq.com/download.html

erlang­18.3­1.el7.centos.x86_64.rpm
socat­1.7.3.2­5.el7.lux.x86_64.rpm
rabbitmq­server­3.6.5­1.noarch.rpm

<!--more-->
#### 安装

```
#安装Erlang
rpm -ivh erlang-18.3-1.el7.centos.x86_64.rpm
#安装RabbitMQ
rpm -ivh socat-1.7.3.2-1.1.el7.x86_64.rpm
rpm -ivh rabbitmq-server-3.6.5-1.noarch.rpm 
```

#### 配置

```
# 开启管理界面
rabbitmq-plugins enable rabbitmq_management
# 修改默认配置信息
vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin/rabbit.app
# 比如修改密码、配置等等，例如：loopback_users 中的 <<"guest">>,只保留guest
```

#### 启动

```
service rabbitmq-server start #启动服务
service rabbitmq-server stop #停止服务
service rabbitmq-server restart #重启服务

设置配置文件
cd /usr/share/doc/rabbitmq-server-3.6.5/
cp rabbitmq.config.example /etc/rabbitmq/rabbitmq.config
如果web控制台无法正常访问考虑安装是否成功以及是防火墙的原因

关闭防火墙
systemctl stop firewalld
```

#### 访问

http://192.168.1.200:15672

guest/guest

![image](/static/img/4.png)