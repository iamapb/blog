---
title: docker-compose
date: 2022-06-06 20:48:51
tags: docker
---
### docker-compose 脚本文档
####  mysql 脚本文档
```xml

version: '3.1'

services:
  mysql:
    container_name: mysql
    image: mysql:5.7
    command: [
        '--character-set-server=utf8',
        '--collation-server=utf8_general_ci',
        '--default-time-zone=+8:00'
        ]
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: soa123456
    volumes:
      - "./db:/var/lib/mysql"
      - "./sql:/tmp/sql"
      - "./init:/docker-entrypoint-initdb.d/"
    ports:
      - "33060:3306"
    networks:
      default:
        ipv4_address: 172.20.0.31

networks:
  default:
    external:
      name: lightnet[root@Server110148 mysql]# 
```
#### elasticsearch
```xml
version: '3'
services:
  elasticsearch:
    image: elasticsearch:6.4.2
    container_name: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
    networks:
      default:
        ipv4_address: 172.20.0.38

volumes:
  esdata:
    driver: local

networks:
  default:
    external:
      name: lightnet[root@Server110148 elasticsearch]# 
```

#### zookeeper
```xml

version: '3.1'
services:
  light-zookeeper:
    image: zookeeper:3.4.10
    restart: always
    container_name: light-zookeeper
    ports:
      - "2181:2181"

networks:
  default:
    external:
      name: lightnet[root@Server110148 zookeeper]# 
```
