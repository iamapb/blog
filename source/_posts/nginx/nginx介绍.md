---
title: nginx介绍
date: 2022-03-13 12:59:05
tags: nginx
--- nginx

### 1 nginx的进行模型
```angular2
1 master进程是用来监听http的请求，用于分发到对应的work进行，
2 work进行是工作进程，work接受master的进行的来源
```
### 2 nginx的web请求处理解析

![](/../../static/nginx/nginx事件处理.png)



