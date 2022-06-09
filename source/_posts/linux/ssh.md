---
title: ssh
date: 2022-03-11 22:20:55
tags: ssh
---

## ssh问题
```xml
Linux SSH命令用了那么久，第一次遇到这样的错误：

ECDSA host key "ip地址" for  has changed and you have requested strict checking
```
## 解决办法
```xml
    ssh-keygen -R "你的远程服务器ip地址"
```

### windows 查询端口号使用情况，并且停止服务
```xml

1、Windows+R启动cmd命令
2、输入命令：netstat -ano ，回车查看系统当前所有端口使用情况
3、查找某一特定端口，输入命令：netstat -ano |findstr “端口号”，回车，可查看被占用端口的应用j进程ID（最后一列）
4、通过应用ID查找应用名称，输入命令：tasklist |findstr “应用进程ID”，可查看应用名称（第一列）
5、通过命令杀掉进程，输入命令：taskkill /f /t /im “进程id或进程名称”

```
