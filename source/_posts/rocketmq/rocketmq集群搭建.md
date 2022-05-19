---
title: rocketmq集群搭建
date: 2021-08-25 23:01:12
tags: rocketmq
---

##本次搭建集群模式为两个master， 两个slave集群， 且复制模式为异步复制。

### 1 集群节点之间关系：

rocketmq1 172.16.28.185 Master1+Slave2 
rocketmq2 172.16.28.186 Master2+Slave1 


### 2. 修改配置文件
rocketMQ 提供了几个模板配置类，在%rocketmq%/conf 目录下。如下：
```xml
drwxr-xr-x. 2 root root   118 Jan  4 05:56 2m-2s-async
drwxr-xr-x. 2 root root   118 Oct 22 13:41 2m-2s-sync
drwxr-xr-x. 2 root root    91 Oct 22 13:41 2m-noslave
drwxr-xr-x. 2 root root    72 Oct 22 13:41 dledger
```

2m-2s-async 是2主2从，异步复制的模板； 2m-2s-sync 是2主2从同步复制的模板； 2m-noslave 是2主没有从的配置。

下面修改 2m-2s-async 里面的配置，修改前将目录复制备份下。

2.1 查看 2m-2s-async 目录下面的文件

broker-a.properties  broker-a-s.properties  broker-b.properties  broker-b-s.properties
四个配置文件构成一个集群，四个配置的brokerClusterName 是一致的，代表一个相同的集群；brokerName 代表大集群中的小集群，也就是主从节点关系。

broker-a.properties、broker-a-s.properties 是成对出现的，一个主一个从， 两者内部的brokerName 一致； 同理，broker-b.properties、broker-b-s.properties 也是如此。

下面修改四个配置文件， 然后后面启动集群的时候选择不同的配置文件启动，然后构成不同的集群。


13.101 机器修改两个配置：

(1) broker-a.properties 文件
```angular2
# 集群名称，整个集群名称
brokerClusterName=DefaultCluster
# 指定master-slave 集群的名称，一个RocketMQ 可以包含多个master-slave 集群
brokerName=broker-a
# brokerId，为0 代表master
brokerId=0
# 指定删除消息存储过期文件的时间为凌晨4点
deleteWhen=04
# 指定未发生更新的消息存储文件的保留时长为48小时，48小时后过期，将会被删除 
fileReservedTime=48
# 指定当前broker为异步复制master 
brokerRole=ASYNC_MASTER
# 指定刷盘策略为异步刷盘
flushDiskType=ASYNC_FLUSH
# 指定Name Server的地址
namesrvAddr=172.16.28.185:9876;172.16.28.186:9876
复制代码
　　增加了个namesrvAddr 地址配置

```

(2) broker-b-s.properties 文件

```angular2
brokerClusterName=DefaultCluster
brokerName=broker-b
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
namesrvAddr=172.16.28.185:9876;172.16.28.186:9876
# 指定Broker对外提供服务的端口，即Broker与producer与consumer通信的端口。默认 10911。由于当前主机同时充当着master1与slave2，而前面的master1使用的是默认 里需要将这两个端口加以区分，以区分出master1与slave2 
listenPort=11911
# 指定消息存储相关的路径。默认路径为~/store目录。由于当前主机同时充当着master1与 slave2，master1使用的是默认路径，这里就需要再指定一个不同路径
storePathRootDir=~/store-s
storePathCommitLog=~/store-s/commitlog
storePathConsumeQueue=~/store-s/consumequeue
storePathIndex=~/store-s/index
storeCheckpoint=~/store-s/checkpoint
abortFile=~/store-s/abort

```
13.102 机器修改两个配置：

(1) broker-b.properties 文件
```angular2
brokerClusterName=DefaultCluster
brokerName=broker-b
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
# 指定Name Server的地址
namesrvAddr=172.16.28.185:9876;172.16.28.186:9876

```

(2) broker-a-s.properties 文件

```angular2
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
namesrvAddr=172.16.28.185:9876;172.16.28.186:9876
# 指定Broker对外提供服务的端口，即Broker与producer与consumer通信的端口。默认 10911。由于当前主机同时充当着master1与slave2，而前面的master1使用的是默认 里需要将这两个端口加以区分，以区分出master1与slave2 
listenPort=11911
# 指定消息存储相关的路径。默认路径为~/store目录。由于当前主机同时充当着master1与 slave2，master1使用的是默认路径，这里就需要再指定一个不同路径 
storePathRootDir=~/store-s
storePathCommitLog=~/store-s/commitlog
storePathConsumeQueue=~/store-s/consumequeue
storePathIndex=~/store-s/index
storeCheckpoint=~/store-s/checkpoint
abortFile=~/store-s/abort

```


### 3. 启动集群

启动： 两个机子启动nameserver

nohup sh bin/mqnamesrv &
启动Broker 集群

(1)rocketmq1 启动 master 节点：
在启动之前需要设置下rocketmq的环境变量,不然就会报以下的错误
![broker无法启动](/../../static/rocketmq/rocketmq-集群1.png)
这是没有设置环境变量。设置环境之后再一次启动服务命令
> nohup sh bin/mqbroker -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-a.properties &


(2)rocketmq2 启动 master 节点：

> nohup sh bin/mqbroker -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-b.properties &

(3)rocket1 启动 slave 节点：

>nohup sh bin/mqbroker -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-b-s.properties &

(4)rocket2 启动 slave 节点：

>nohup sh bin/mqbroker -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-a-s.properties &

4. 两个机子查看java 进程
```angular2
[root@rocketmq1 rocketmq-4.9.2]# jps -l | grep -v 'jps'
29170 org.apache.rocketmq.broker.BrokerStartup
29097 org.apache.rocketmq.broker.BrokerStartup
15210 org.apache.rocketmq.namesrv.NamesrvStartup
rocketmq2:

[root@rocketmq2 rocketmq-4.9.2]# jps -l | grep -v 'jps'
15697 org.apache.rocketmq.namesrv.NamesrvStartup
25251 org.apache.rocketmq.broker.BrokerStartup
25335 org.apache.rocketmq.broker.BrokerStartup

```

5. 查看集群

(1) mqadmin管理工具可以查看，mqadmin 是自带的一个管理工具。 关于mqadmin 具体用法可以参考上面连接。
![集群](/../../static/rocketmq/rocketmq-集群2.png)

             
(2) mqadmin 查看相关topic
```angular2
[root@rocketmq1 rocketmq-4.9.2]# ./bin/mqadmin topicList -n 172.16.28.185:9876
RocketMQLog:WARN No appenders could be found for logger (io.netty.util.internal.InternalThreadLocalMap).
RocketMQLog:WARN Please initialize the logger system properly.
RMQ_SYS_TRANS_HALF_TOPIC
%RETRY%please_rename_unique_group_name_4
BenchmarkTest
OFFSET_MOVED_EVENT
TBW102
SELF_TEST_TOPIC
DefaultCluster
SCHEDULE_TOPIC_XXXX
DefaultCluster_REPLY_TOPIC
broker-b
TopicTest
broker-a
%RETRY%__MONITOR_CONSUMER
%RETRY%TOOLS_CONSUMER
localhost.localdomain

```
(3) 查看Topic的路由信息
```angular2
[root@rocketmq2 rocketmq-4.9.2]# ./bin/mqadmin topicRoute -t TopicTest -n 172.16.28.185:9876
RocketMQLog:WARN No appenders could be found for logger (io.netty.util.internal.InternalThreadLocalMap).
RocketMQLog:WARN Please initialize the logger system properly.
{
        "brokerDatas":[
                {
                        "brokerAddrs":{0:"172.16.28.186:10911",1:"172.16.28.185:11911"
                        },
                        "brokerName":"broker-b",
                        "cluster":"DefaultCluster"
                },
                {
                        "brokerAddrs":{0:"172.16.28.185:10911",1:"172.16.28.186:11911"
                        },
                        "brokerName":"broker-a",
                        "cluster":"DefaultCluster"
                }
        ],
        "filterServerTable":{},
        "queueDatas":[
                {
                        "brokerName":"broker-a",
                        "perm":6,
                        "readQueueNums":4,
                        "topicSysFlag":0,
                        "writeQueueNums":4
                },
                {
                        "brokerName":"broker-b",
                        "perm":6,
                        "readQueueNums":4,
                        "topicSysFlag":0,
                        "writeQueueNums":4
                }
        ]
}

```
