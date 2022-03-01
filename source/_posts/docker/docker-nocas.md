---
title: docker 运行脚本
date: 2021-03-09 15:16:52
tags: docker
---
1 docker 运行nacos
```xml
    docker run -d --restart=always --name nacos -p 8848:8848 -p 20880:20880 --net=host -e PREFER_HOST_MODE=hostname -e MODE=standalone nacos/nacos-server
```

2 docker 运行mysql
```xml
    #!/bin/sh
    
    mysql_password=318412mmpb
    docker_image=mysql:5.7
    docker_container=mysql
    
    docker rm -f $docker_container
    docker run -d --restart=always \
        --name $docker_container \
        -e MYSQL_ROOT_PASSWORD=$mysql_password  \
        -v /opt/data/mysql:/var/lib/mysql \
        -p 3306:3306 \
        $docker_image

```
3 docker 运行phpmyadmin (包含服务器)
```xml
    #!/bin/sh
    
    set -e
    set -o pipefail
    
    docker_bridge_gateway=`ifconfig | grep "inet " | grep -Fv 127.0.0.1 | awk '{print $2}' | head -n 1`
    echo $docker_bridge_gateway
    
    docker_image=phpmyadmin/phpmyadmin:4.8.5
    docker_container=pma
    port=9001:80
    host1=db:$docker_bridge_gateway
    
    docker rm -f $docker_container || true
    docker run -d --restart=always \
        --name=$docker_container \
        --add-host=$host1 \
        -p $port \
        -e PMA_ARBITRARY=1 \
        $docker_image
    
 
    
```

4 docker 运行phpmyadmin（用户密码）

```xml
    #!/bin/sh    
    
    docker_image=phpmyadmin/phpmyadmin
    docker_container=pma
    
    docker rm -f $docker_container
    docker run -d --restart=always \
        --name $docker_container  \
        --link mysql:db \
        -p 8001:80  \
        -v /opt/data/phpmyadmin/config.user.inc.php:/etc/phpmyadmin/config.user.inc.php \
        $docker_image
```

5 docker 获取https 命令
```xml
    #!/bin/sh
    
    set -e
    set -o pipefail
   
    
    mkdir -p /var/log/letsencrypt
    
    docker run -it --rm --name certbot \
                -v "/etc/letsencrypt:/etc/letsencrypt" \
                -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
                -v "/var/log/letsencrypt:/var/log/letsencrypt" \
                -p 80:80 \
                -p 443:443  \
                 certbot/certbot certonly --standalone -m 1174437797@qq.com  \ // 这里是你的邮箱
                  -d www.iamapb.com  \  // 这里是你的域名 可以添加多个
    
    cd /etc/letsencrypt/live/www.iamapb.com/
    cat cert.pem chain.pem privkey.pem > prod.pem
    /usr/bin/cp -f prod.pem /etc/haproxy/prod.pem


```

```xml
docker 删除镜像失败两种情况
1 原因:
    容器还存在是无法删除镜像的

2 原因
  容器不存在，删除镜像时候提示 No such image
   docker 镜像文件是存在/var/lib/docker/image/overlay2/imagedb/content/sha256目录下的文件即可    
```
