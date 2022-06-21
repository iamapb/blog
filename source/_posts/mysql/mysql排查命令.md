---
title: mysql排查命令
date: 2022-06-20 16:27:16
tags: mysql
---
## 部分mysql问题排查命令
 1 show processlist;
![](../../static/mysql/show.png)
```xml
    用于查看具体哪些sql执行比较慢 
```

2 show open tables where in_use >0;
```xml
    用户查看现在有多少表正在使用中

````

3 
