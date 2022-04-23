---
layout: redis
title: redis使用
date: 2022-04-23 15:55:47
tags: redis

```xml
tar -zxvf redis-5.0.5.tar.gz

安装gcc编译环境，如果已经安装过了，那么就是 nothing to do

yum install gcc-c++ 
                                                            
进入到 redis-5.0.5 目录，进行安装：

make && make install                                                            

配置redis，在utils下，拷贝redis_init_script到/etc/init.d目录，目的要把redis作为开机自启动

创建 /usr/local/redis，用于存放配置文件 redis.conf

拷贝到 /usr/local/redis 下

修改redis.conf这个核心配置文件

修改 daemonize no -> daemonize yes，目的是为了让redis启动在linux后台运行

建议修改为： /usr/local/redis/working，名称随意
修改如下内容，绑定IP改为 0.0.0.0 ，代表可以让远程连接，不收ip限制
最关键的是密码，默认是没有的，一定要设置
修改 redis_init_script 文件中的redis核心配置文件为如下：
CONF="/usr/local/redis/redis.conf"

为redis启动脚本添加执行权限，随后运行启动redis：

chmod 777 redis_init_script

./redis_init_script

并且修改redis核心配置文件名称为：6379.conf

设置redis开机自启动，修改 redis_init_script，添加如下内容

#chkconfig: 22345 10 90
#description: Start and Stop redis


chkconfig redis_init_script on
```

### 2 redis 主从复制的配置修改
1 修改配置文件
```xml
主要修改配置文件的三个地方
  replicaof 主的ip 端口
  
  masterauth 主密码
  
  replica-read-only yes
  
```
2 重启服务器

