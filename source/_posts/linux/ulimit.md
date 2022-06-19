---
title: ulimit
date: 2022-06-17 17:25:33
tags: linux
---
## ulimit 命令
###  1 问题
```xml
    最近在工作当中，某个服务挂了 不能使用，服务日志显示
    unable to create new native thread 无法创建新的线程
    
    原因：

   1.应用程序创建了太多的线程，超过了系统承载。

   2.服务器不允许你的应用创建这么多线程。linux用户除了Root以外默认创建线程数是1024.
```

### 2 排除问题
```xml
   1 ulimit -a 查看系统的具体设置的参数
    ![](/../../static/linux/ulimit.png)
    系统设置的用户数较少
```

