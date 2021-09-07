---
title: nginx-ha keepalived 高可用主备
date: 2021-09-04 17:29:34
tags:
---

## 1 keepalived 安装部署

```xml
 1 下载软件
 2 tar -zxvf keepalived-2.0.18.tar.gz
 3 进入目录 ./configure --prefix=/usr/local/keepalived --sysconf=/etc
 4 错误  : Please install libnl/libnl-3 dev libraries to support IPv6 with IPVS.
                                                                                             
 5.2. 安装libnl/libnl-3依赖
 
 yum -y install libnl libnl-devel  

                                                                                            
 5.3. 重新configure一下，此时OK。
 
 安装keepalived
 
 make && make install
 1
                                                                                            
 进入到/etc/keepalived，该目录下为keepalived核心配置文件

    

```
## 2 配置keepalived - 主
```xml
. 通过命令 vim keepalived.conf 打开配置文件
global_defs {
   # 路由id：当前安装keepalived的节点主机标识符，保证全局唯一
   router_id keep_171
}

vrrp_instance VI_1 {
    # 表示状态是MASTER主机还是备用机BACKUP
    state MASTER
    # 该实例绑定的网卡
    interface ens33
    # 保证主备节点一致即可
    virtual_router_id 51
    # 权重，master权重一般高于backup，如果有多个，那就是选举，谁的权重高，谁就当选
    priority 100
    # 主备之间同步检查时间间隔，单位秒
    advert_int 2
    # 认证权限密码，防止非法节点进入
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 虚拟出来的ip，可以有多个（vip）
    virtual_ipaddress {
        192.168.1.161
    }
}
```

## 3 keepalived 配置nginx自动重启

```xml

      

3-10 附：Keepalived配置Nginx自动重启
Keepalived配置Nginx自动重启
1. 增加Nginx重启检测脚本
    vim /etc/keepalived/check_nginx_alive_or_not.sh

--------                            
#!/bin/bash

A=`ps -C nginx --no-header |wc -l`
# 判断nginx是否宕机，如果宕机了，尝试重启
if [ $A -eq 0 ];then
    /usr/local/nginx/sbin/nginx
    # 等待一小会再次检查nginx，如果没有启动成功，则停止keepalived，使其启动备用机
    sleep 3
    if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
        killall keepalived
    fi
fi

----------------
                                                                                            
增加运行权限

chmod +x /etc/keepalived/check_nginx_alive_or_not.sh

2. 配置keepalived监听nginx脚本
vrrp_script check_nginx_alive {
    script "/etc/keepalived/check_nginx_alive_or_not.sh"
    interval 2 # 每隔两秒运行上一行脚本
    weight 10 # 如果脚本运行成功，则升级权重+10
    # weight -10 # 如果脚本运行失败，则升级权重-10
}


                                                            
3. 在vrrp_instance中新增监控的脚本
track_script {
    check_nginx_alive   # 追踪 nginx 脚本
}

                                                                                            
4. 重启Keepalived使得配置文件生效
systemctl restart keepalived
```

## 4 配置Keepalived双主热备
```xml
  


配置Keepalived双主热备
规则：以一个虚拟ip分组归为同一个路由

主节点配置：
global_defs {
   router_id keep_171
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.161
    }
}

vrrp_instance VI_2 {
    state BACKUP
    interface ens33
    virtual_router_id 52
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.162
    }
}

                                代码块                                                            
备用节点配置
global_defs {
   router_id keep_172
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.161
    }
}

vrrp_instance VI_2 {
    state MASTER
    interface ens33
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.162
    }
}
                                代码块                                                            
别忘记分别重启两条Keepalived
# 重启Keepalived
systemctl restart keepalived

```
