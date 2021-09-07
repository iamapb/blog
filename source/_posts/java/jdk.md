---
title: jdk
date: 2021-02-05 17:08:10
tags: jdk
---

### linux 虚拟机 wget 跳过验证下载jdk1.8

> 现在记录使用wget 跳过验证下载jdk 非常实用
```java
  wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"
```
    
