---
title: jar包 执行
date: 2021-03-09 15:00:04
tags: java
---
### Linux执行Jar包： no main manifest attribute 

Linux 执行jar包的时候， 报这个错误： no main manifest attribute

![](/../../static/java/jar.png)

原来pom.xml中少了如下配置
```xml
    <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <configuration>
                        <executable>true</executable>
                    </configuration>
                </plugin>
            </plugins>
        </build>
   
```
