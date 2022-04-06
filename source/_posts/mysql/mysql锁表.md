---
title: mysql锁表
date: 2022-04-06 15:03:20
tags: mysql
---
###  1.业务场景: 
某个页面的查询列表比较慢，有时候会报服务器异常 500。 在排除网络稳定，sql语句没问题的情况下，在看看是不是别的问题，猜测是不是锁表的问题。
锁表: 

### 2 排查原因:

查询是否锁表
show OPEN TABLES where In_use > 0;
![](/../../static/mysql/mysql锁表.jpg)
