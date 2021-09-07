---
title: nginx配置https
date: 2021-02-23 13:54:15
tags: nginx
---
#### centos7下 使用Certbot+nginx 配置https环境
> Let's Encrypt是一个为网站提供免费的SSL/TLS证书的机构。官方推荐使用Certbot工具进行签发。Certbot可自动签发Let's Encrypt证书，但证书有效期只有三个月，可以通过配置定时任务进行自动续期，以此实现永久生效的https环境。
  本文使用Certbot+Nginx进行单域名和泛域名的https环境搭建。
  单域名证书：只能保护一个域名，可以是顶级域名也可以是二级域名
  泛域名证书：也叫通配符证书，可以保护一个域名及该域名下所有二级域名，不限制下级域名数量
  

### 1 安装Certbot 
> yum install certbot python2-certbot-nginx

### 2 安装单域名证书
 
#### 2.1 会自动下载证书并配置nginx (后续去nginx配置可以查看) 
> certbot --nginx

#### 2.2  自动下载证书 自己手动配置 
> certbot certonly --nginx
