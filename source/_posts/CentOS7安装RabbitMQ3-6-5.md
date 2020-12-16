---
title: CentOS7安装RabbitMQ3.6.5
toc: true
abbrlink: 64fadf5f
date: 2020-12-13 20:31:56
tags: CentOS
categories: Linux运维
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
# 比如修改密码、配置等等，例如：loopback_users 中的 <<"guest">>,只保留guest用户
将{loopback_users, [<<"guest">>]},改为{loopback_users, [guest]}
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


#### 问题

- error: Failed dependencies: tcp_wrappers is needed by socat-1.7.3.2-1.1.el7.x86_64

```
[root@localhost rabbitmq]# rpm -ivh socat-1.7.3.2-1.1.el7.x86_64.rpm 
warning: socat-1.7.3.2-1.1.el7.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID 87e360b8: NOKEY
error: Failed dependencies:
	tcp_wrappers is needed by socat-1.7.3.2-1.1.el7.x86_64
	
#在rpm 语句后面加上 --force --nodeps 即原本为 rpm -ivh *.rpm 现在改成 rpm -ivh *.rpm --force --nodeps就可以了。
#nodeps的意思是忽视依赖关系。因为各个软件之间会有多多少少的联系。有了这两个设置选项就忽略了这些依赖关系，强制安装或者卸载。
rpm -ivh socat-1.7.3.2-1.1.el7.x86_64.rpm --force --nodeps
```

- guest登录不上Login failed

```
#查看日志
tail -n50 /var/log/rabbitmq/rabbit\@localhost.log

=WARNING REPORT==== 14-Dec-2020::10:38:21 ===
HTTP access denied: user 'guest' - User can only log in via localhost

#注:  默认用户guest 只允许localhost登录。
#创建远程登录用户,guest用户只能在localhost下运行:
rabbitmqctl add_user admin admin
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```