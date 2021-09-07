---
title: es练习
date: 2021-04-19 22:40:30
tags: es
---

## 1 es 核心术语
```xml
    es            mysql
    索引 index     表
    文档 document 行
    字段 fields   列 

```
## 2 安装es
```xml
 1 修改核心配置文件 elasticesarch.yml
  1.1 修改es节点名称
  ![](/../../static/es/es-1.jpg)
  ![](/../../static/es/es-2.jpg)
  1.2 修改data数据地址和日志地址
  ![](/../../static/es/es-3.jpg)
  1.3 绑定es网络ip 和端口号
  ![](/../../static/es/es-4.jpg)
  1。4 修改jvm参数
  ![](/../../static/es/es-5.jpg)
  1.5 es 不允许root用户操作 需要添加用户
  useradd esuser
  chown -R esuser: esuser /usr/local/elasticsearch 进行授权
  su esuser
  whoami
  
  1.6 启动报错是因为没有修改参数 切换root用户
  vim /etc/security/limits.conf
  * soft nofile 65536
  * hard nofile 131072
  * soft nproc 2048
  * hard nproc 4096
  
  vim /etc/sysctl.conf
  vm.max_map_count=262145
  
  sysctl -p 刷新一下
  
  
  
  
  
  
  
 
 
 
 
   

```
