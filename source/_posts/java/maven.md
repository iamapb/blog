---
title: maven   引入aliyun的短信回执外部的jar包
date: 2021-02-02 15:19:37
tags: maven
---

### maven 引入外部jar包并且打包

 在使用aliyun 进行短信消息回执开发的时候，maven是无法引入对应的jar的 需要进行下载并且引入到项目中去
 
 + 首先下载对应的jar包 在项目中创建一个对应的lib 存放对应的jar包
 ![图片](/../../static/maven/maven.png)
 
 + 在对应项目的pom文件下面引入对应的jar包
 + project.basedir 是项目的目录定死的
 
 ```xml
  <dependencies>
       <dependency>
           <groupId>com.aliyun</groupId>
           <artifactId>alicom-sdk-mns</artifactId>
           <version>1.1.8</version>
           <scope>system</scope>
           <systemPath>${project.basedir}/lib/aliyun-sdk-mns-1.1.8.jar</systemPath>
       </dependency>
       <dependency>
           <groupId>org.apache.http</groupId>
           <artifactId>httpasyncclient</artifactId>
           <version>4.1</version>
           <scope>system</scope>
           <systemPath>${project.basedir}/lib/httpasyncclient-4.1.jar</systemPath>
       </dependency>
       <dependency>
           <groupId>org.apache.http</groupId>
           <artifactId>httpcore</artifactId>
           <version>4.4.1</version>
           <scope>system</scope>
           <systemPath>${project.basedir}/lib/httpcore-4.4.1.jar</systemPath>
       </dependency>
       <dependency>
           <groupId>org.apache.http</groupId>
           <artifactId>httpcore-nio</artifactId>
           <version>4.4.1</version>
           <scope>system</scope>
           <systemPath>${project.basedir}/lib/httpcore-nio-4.4.1.jar</systemPath>
       </dependency>
       <dependency>
           <groupId>org.apache.log4j</groupId>
           <artifactId>log4j</artifactId>
           <version>1.2.17</version>
           <scope>system</scope>
           <systemPath>${project.basedir}/lib/log4j-1.2.17.jar</systemPath>
       </dependency>
  </dependencies>
 ```
 
 + 在build 里面添加下面的一句话
 
  
    ```java
        <build>
               <plugins>
                    <plugin>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-maven-plugin</artifactId>
                        <configuration>
                            <includeSystemScope>true</includeSystemScope>
                        </configuration>
                    </plugin>
                </plugins>
         </build>
    ```
 


