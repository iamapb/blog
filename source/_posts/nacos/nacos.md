---
title: nacos知识点
date: 2022-07-12 09:06:27
tags: nacos
---

### 1 nacos寻址方式
地址服务器寻址(推荐)
> Nacos 官方推荐的⼀种集群成员节点信息管理，该模式利用了⼀个简易的 web 服务器，用于管理 cluster.conf 文件的内容信息

![](/../../static/nacos/nacos1.png)

2 文件寻址(默认)
>文件寻址模式是 Nacos 集群模式下的默认寻址实现。文件寻址模式很简单，其实就是每个 Nacos 节点需要维护⼀个叫做 cluster.conf 的文件

缺点: 运维成本大，集群捡陈冠节点列表的数据会不一致

3 单机寻址
> 找到自己的 IP:PORT 组合信息，然后格式化为⼀个节点信息， 调用 afterLookup 然后将信息存储到 ServerMemberManager 中

### 2 nacos数据模型
>是⼀种服务-集群-实例的三层模型,满足服务在所有场景下的数据存储和管理
