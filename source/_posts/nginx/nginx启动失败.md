---
title: nginx启动失败
date: 2021-02-20 21:13:06
tags: nginx
---
### nginx 启动失败解决方案

> 1服务器重启中后， 执行 nginx t 是ok的， 然而在执行法nginx -s reload 的时候报错
```xml
   nginx: [error] open() "/run/nginx.pid" failed (2: No such file or directory)
```
在对应的目录创建对应缺少的文件 mkdir /run.nginx.pid

> 2 继续执行 nginx -s reload 出现一下错误
```xml
  nginx: [error] invalid PID number "" in "/run/nginx.pid"
```
表示pid是无效的 可以使用./nginx -h 查看对应的信息

-c filename   : set configuration file (default: conf/nginx.conf)

执行如下命令就能解决对应的问题
```xml
    nginx -c /etc/nginx/nginx.conf
```

