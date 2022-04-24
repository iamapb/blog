---
title: rocketmq安装
date: 2022-04-08 13:56:00
tags: rocketmq
---
### 安装步骤 windows 
1 下载 https://rocketmq.apache.org/dowloading/releases/
2 修改启动的参数 vim runserver.sh , runbroker.sh  conf/broker.conf 这个是ip地址修改
3 启动
```xml
cd bin/ 
启动 nameservice start mqnamesrv.cmd
启动 broker start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true


```
