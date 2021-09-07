---
title: rabbitmq
date: 2021-04-23 22:29:57
tags: rabbitmq
---
## 安装mq
1 rpm
```xml
   brew install rabbitmq 

   可以直接下 然后打开浏览器 localhost:15672 用户名guest guest
```

2
```xml

### RabbitMQ急速入门
  

 ## 1. 首先在Linux上进行一些软件的准备工作，yum下来一些基础的软件包

yum install build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel m4 ncurses-devel tk tc xz

## 配置好主机名称：/etc/hosts /etc/hostname

## 2. 下载RabbitMQ所需软件包（本神在这里使用的是 RabbitMQ3.6.5 稳定版本）

wget www.rabbitmq.com/releases/erlang/erlang-18.3-1.el7.centos.x86_64.rpm
wget http://repo.iotti.biz/CentOS/7/x86_64/socat-1.7.3.2-1.1.el7.lux.x86_64.rpm
wget www.rabbitmq.com/releases/rabbitmq-server/v3.6.5/rabbitmq-server-3.6.5-1.noarch.rpm

## 3. 安装服务命令

rpm -ivh erlang-18.3-1.el7.centos.x86_64.rpm 
rpm -ivh socat-1.7.3.2-1.1.el7.x86_64.rpm
rpm -ivh rabbitmq-server-3.6.5-1.noarch.rpm

## 4. 修改用户登录与连接心跳检测，注意修改

vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin/rabbit.app
修改点1：loopback_users 中的 <<"guest">>,只保留guest （用于用户登录）
修改点2：heartbeat 为10（用于心跳连接）

## 5. 安装管理插件

## 5.1 首先启动服务(后面 | 包含了停止、查看状态以及重启的命令)

/etc/init.d/rabbitmq-server start | stop | status | restart

## 5.2 查看服务有没有启动： lsof -i:5672 （5672是Rabbit的默认端口）

rabbitmq-plugins enable rabbitmq_management

## 5.3 可查看管理端口有没有启动： 

lsof -i:15672 或者 netstat -tnlp | grep 15672

## 6. 一切OK 我们访问地址，输入用户名密码均为 guest ：

## http://你的ip地址:15672/

## 7. 如果一切顺利，那么到此为止，我们的环境已经安装完啦


```

### 3  如何保证消息的100%的投递(一)
1 保证消息的成功发出
2 保证mq节点的成功接受
3 发送端收到mq节点的确认应答
4 完善的消息进行补偿

解决方案 消息的落库 对消息的状态进行打标

![](/../../static/mq/消息落库.png)

##1.如何保证消息没有重复消费

方案: 消费端实现幂等性 意味着 我们的消息永远不会消费多次
利用数据库的主键唯一性进行去重复 

##2.confirm 确认消息 流程解析
![](/../../static/mq/confirm确认消息.png)

如何实现confirm 消息
1 在channel上开启确认模式 channel.confirmSelect()
2 在channel上面添加监听时间 addConfirmListener 监听成功和失败的返回结果, 进行后续的处理

## 3 return 消息
![](/../../static/mq/return机制.pmg)

## 3 消费端的流控
rabbitmq 


