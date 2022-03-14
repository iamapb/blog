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
