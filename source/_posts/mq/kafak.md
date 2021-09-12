---
title: kafak
date: 2021-05-16 15:15:18
tags: mq
---

## 1.kafka 高性能的原因
> 1 顺序写，Page Cache 空中接力，高效读写
  2 高性能高吞吐
  3 后台异步，主动Flush
  4 预读策略 IO
  
### 2 pageCache 
```xml
    操作系统实现的一种磁盘缓存的一种策略 减少磁盘的io 把数据放入内存里面

```   

### 3 zeroCopy 
```xml

```
  
## 2 特性:
> 1 分布式 partition 分区的特点
  2 跨平台
  3 实时性
  4 伸缩性  
  
## kafka 消费者相关内容  
