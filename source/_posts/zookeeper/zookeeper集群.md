---
title: zookeeper集群
date: 2021-09-14 21:14:48
tags: zookeeper
---
## 1.Zookeeper集群环境搭建：

```shell
1. 准备工作：
## 准备3个节点，要求配置好主机名称，服务器之间系统时间保持一致
## 注意 /etc/hostname 和 /etc/hosts 配置主机名称（在这个里我准备bhz221,bhz222,bhz223三节点）
## 特别注意 以下操作3个节点要同时进行操作哦！
## 注意关闭防火墙
1.启动防火墙systemctl start firewalld
2.关闭防火墙systemctl stop firewalld
3.重启防火墙systemctl restart firewalld
4.查看防火墙状态systemctl status firewalld
5.开机禁用防火墙systemctl disable firewalld
## 本地访问：ping


2. 上传zk到三台服务器节点
## 注意我这里解压到/usr/local下
## 2.1 进行解压： 
cd /usr/local/software
tar -zxvf zookeeper-3.4.6.tar.gz -C /usr/local/
## 跳转到cd /usr/local/
cd ..

## 2.2 修改环境变量： vim /etc/profile

## 这里要添加zookeeper的全局变量
export JAVA_HOME=/usr/local/jdk1.8
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.4.6

export PATH=.:$JAVA_HOME/bin:$ZOOKEEPER_HOME/bin:$PATH

## 2.3 刷新环境变量： 
source /etc/profile

## 2.4 到zookeeper下修改配置文件： 
## 2.5.1 首先到指定目录：
cd /usr/local/zookeeper-3.4.6/conf
## 2.5.2 然后复制zoo_sample.cfg文件，复制后为zoo.cfg：
mv zoo_sample.cfg zoo.cfg
## 2.5.3 vim zoo.cfg 然后修改两处地方, 最后保存退出：
vim /usr/local/zookeeper-3.4.6/conf/zoo.cfg
(1) 修改数据的dir
dataDir=/usr/local/zookeeper-3.4.6/data
(2) 修改集群地址
server.0=bhz221:2888:3888
server.1=bhz222:2888:3888
server.2=bhz223:2888:3888

## 2.5.4 增加服务器标识配置，需要2步骤，第一是创建文件夹和文件，第二是添加配置内容： 
(1) 创建文件夹： 
mkdir /usr/local/zookeeper-3.4.6/data
(2) 创建文件myid 路径应该创建在/usr/local/zookeeper-3.4.6/data下面，如下：
vim /usr/local/zookeeper-3.4.6/data/myid

## 2.5.5 注意这里每一台服务器的myid文件内容不同，分别修改里面的值为0，1，2；
## 与我们之前的zoo.cfg配置文件里：server.0，server.1，server.2 顺序相对应，然后保存退出；

## 2.6 到此为止，Zookeeper集群环境大功告成！启动zookeeper命令
启动路径：/usr/local/zookeeper-3.4.6/bin（也可在任意目录，因为配置了环境变量）
执行命令：zkServer.sh start (注意这里3台机器都要进行启动，启动之后可以查看状态)
查看状态：zkServer.sh status (在三个节点上检验zk的mode, 会看到一个leader和俩个follower)
集群关闭：zkServer.sh stop 
	
## zkCli.sh 进入zookeeper客户端

根据提示命令进行操作： 
查找：ls /   ls /zookeeper
创建并赋值： create /imooc zookeeper
获取： get /imooc 
设值： set /imooc zookeeper1314 
PS1: 任意节点都可以看到zookeeper集群的数据一致性
PS2: 创建节点有俩种类型：短暂（ephemeral） 持久（persistent）, 这些小伙伴们可以查找相关资料，我们这里作为入门不做过多赘述！
```

开机启动：

```shell
cd /etc/rc.d/init.d/
touch zookeeper
chmod 777 zookeeper
vim zookeeper

开机启动zookeeper脚本：


#!/bin/bash

#chkconfig:2345 20 90
#description:zookeeper
#processname:zookeeper
export JAVA_HOME=/usr/local/jdk1.8
export PATH=$JAVA_HOME/bin:$PATH
case $1 in
          start) /usr/local/zookeeper-3.4.6/bin/zkServer.sh start;;
          stop) /usr/local/zookeeper-3.4.6/bin/zkServer.sh stop;;
          status) /usr/local/zookeeper-3.4.6/bin/zkServer.sh status;;
          restart) /usr/local/zookeeper-3.4.6/bin/zkServer.sh restart;;
          *)  echo "require start|stop|status|restart"  ;;
esac

开机启动配置：chkconfig zookeeper on

验证：
chkconfig --add zookeeper
chkconfig --list zookeeper

这个时候我们就可以用servicezookeeper start/stop来启动停止zookeeper服务了

使用chkconfig--add zookeeper命令把zookeeper添加到开机启动里面

添加完成之后接这个使用chkconfig--list 来看看我们添加的zookeeper是否在里面

如果上面的操作都正常的话；你就可以重启你的linux服务器了
```

## 2 zookeeper说明


