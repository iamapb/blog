---
title: kafak 概念
date: 2021-05-16 15:15:18
tags: kafka
---

## 1.kafka 高性能的原因
> 1 顺序写，Page Cache 空中接力，高效读写
  2 高性能高吞吐
  3 后台异步，主动Flush
  4 预读策略 IO
  
### 1.1 pageCache 
```xml
    操作系统实现的一种磁盘缓存的一种策略 减少磁盘的io 把数据放入内存里面

```   

### 1.2 zeroCopy 
```xml

```
  
## 2 特性:
> 1 分布式 partition 分区的特点
  2 跨平台
  3 实时性
  4 伸缩性  
  
## 3 kafka 相关概念:
   topic: 一个
   partition: 
   replica
   ISR: kafka的follow副本的拉取集合，拉去的leader的数据在同步的时间容忍度的范围内， 能够准确的同步leader的数据
   OSR: kafka的follow副本数据集合，拉取leader的数据是超出了时间的范围，不能够及时的同步leader的数据，获取的不是实时的数据
   HW: heigh watermark 高水位线，消费者最多能够拉取到高水位线的消息
   LEO: Log End Offset, 日志文件的最后一条记录的offset(偏移量)
   ISR集合与HW 和 LEO有关系
   ![](/../../static/kafka/isr.png)
   

## 4kafka集群环境搭建

- kafka环境搭建准备：

  ```shell
  - 准备zookeeper环境（zookeeper-3.4.6）
  - 下载kafka安装包：https://archive.apache.org/dist/kafka/2.1.0/kafka_2.12-2.1.0.tgz
  - 上传到：192.168.11.221
  ```

- 搭建kafka环境：

  ```shell
  ## 1 解压kafka_2.12-2.1.0.tgz.gz包到/usr/local/
  tar -zxvf kafka_2.12-2.1.0.tgz.gz -C /usr/local/
  
  ## 2 重命名kafka_2.12
  cd /usr/local
  mv kafka_2.12-2.1.0 kafka_2.12
  
  ## 3 修改kafka配置文件
  vim /usr/local/kafka_2.12/config/server.properties
  ## 修改内容：
  ## The id of the broker. This must be set to a unique integer for each broker
  broker.id=0
  port=9092
  host.name=192.168.11.221
  dvertised.host.name=192.168.11.221
  log.dirs=/usr/local/kafka_2.12/kafka-logs
  num.partitions=5
  zookeeper.connect=192.168.11.221:2181,192.168.11.222:2181,192.168.11.223:2181
  
  ## 4 创建kafka存储消息（log日志数据）的目录
  mkdir /usr/local/kafka_2.12/kafka-logs
  
  ## 5 到此为止，kafka已经配置成功，执行启动命令，启动kafka
  /usr/local/kafka_2.12/bin/kafka-server-start.sh /usr/local/kafka_2.12/config/server.properties &
  
  ## 6 安装kafka manager可视化管控台：把jar包（kafka manager）上传到 192.168.11.222
  ## 6.1 解压zip文件
  unzip kafka-manager-2.0.0.2.zip -d /usr/local/
  
  ## 6.2 修改配置文件：
  vim /usr/local/kafka-manager-2.0.0.2/conf/application.conf
  ## 修改内容：
  kafka-manager.zkhosts="192.168.11.221:2181,192.168.11.222:2181,192.168.11.223:2181"
  
  ## 6.3 192.168.11.222节点启动kafka manager 控制台
  /usr/local/kafka-manager-2.0.0.2/bin/kafka-manager &
  
  ## 6.4 浏览器访问控制台：默认端口号是9000
  http://192.168.11.222:9000/
  
  ## 6.5 添加Cluster集群
  
  ## 7 集群验证：
  ## 7.1 通过控制台创建了一个topic为"test" 2个分区 1个副本
  
  ## 7.2 消费发送与接收验证
  cd /usr/local/kafka_2.12/bin
  ## 启动发送消息的脚本
  ## --broker-list 192.168.11.221 指的是kafka  broker的地址列表
  ##  --topic test 指的是把消息发送到test主题
  kafka-console-producer.sh --broker-list 192.168.11.221:2181 --topic test
  
  ## 启动接收消息的脚本
  kafka-console-consumer.sh --bootstrap-server 192.168.11.221:9092 --topic test
  
  
  
  ```

- kafka参数配置

  ```shell
  ## The number of threads that the server uses for receiving requests from the network and sending responses to the network
  num.network.threads=3
  
  # The number of threads that the server uses for processing requests, which may include disk I/O
  num.io.threads=8
  
  
  
  ```
- 

## 5 kafka 生产者核心概念 producerRecord 
   1.1发送消息体: Class ProducerRecord<K, V> {
   >    String topic;
        Integer partition;
        Headers headers;
        K key;
        V value;
        Long timestamp;
           
   1.2 必要的参数配置:
   >    bootstrap.servers: ip 节点集群
        key.serializer     二进制的消息进行发送和接收
        value.serializer   
        client.id         生产者的id
        Producer.Config
         
   1.3 kafkaProducer是线程安全的 
   1.4 kafka消息发送的重试机制   retires 参数 重复发送
   1.4 kafka生产者重要的参数     
   > acks: 指定发送消息后，Broker端至少有多少个副本接收到该消息:默认为 acks=1;
     acks=1; 主副本接收到消息了
     acks=0; 生产者发送消息之后不需要等待任何服务器端的响应;
     acks=-1 acks=all 生产者在发送消息之后，需要等待ISR中的所有副本都成功写入消息之后才能够收到来自服务端的成功响应

   1.5 acks=-a or acks=all 一定能够保证消息的可靠性投递吗 不一定 需要看min.insync.replaices 的值

   max.request.size: 用来限制生产者客户端能够发送的消息的最大值
   retries 和 retry.backoff.msretries 重试的次数和重试的间隔
   batch.size 一次性批量发送的数据
   buffer.memory: 缓存提升性能的参数

     
## 6 kafka消费者
   6.1 KafkaConsumer是线程非安全的,在kafkaConsumer中定义了一个acquire方法用来检测是否只有一个线程在操作，如果
   有其他的线程操作则会抛出ConcurrentModifactionException，用来检测是否线程安全    
   6.2 kafka的消费模型 点对点和发布订阅模型的区分是根据  消费者组 这个概念来区分的
      点对点的话 是根据不同的消费组
      发布订阅模型的话 是同属于一个组 不同的消费者
              
            
         
    
    




   
  
