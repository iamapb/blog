---
title: nginx配置https
date: 2021-02-23 13:54:15
tags: nginx
---
#### 1 centos7下 使用Certbot+nginx 配置https环境
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

## 2 腾讯云配置

1. 安装SSL模块
要在nginx中配置https，就必须安装ssl模块，也就是: http_ssl_module。
```xml

进入到nginx的解压目录： /home/software/nginx-1.16.1

新增ssl模块(原来的那些模块需要保留)

./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi  \
--with-http_ssl_module
```



2 编译和安装
```
make
make install
```


3. 配置HTTPS
```xml

把ssl证书 *.crt 和 私钥 *.key 拷贝到/usr/local/nginx/conf目录中。

新增 server 监听 443 端口：

server {
    listen       443;
    server_name  www.imoocdsp.com;

    # 开启ssl
    ssl     on;
    # 配置ssl证书
    ssl_certificate      1_www.imoocdsp.com_bundle.crt;
    # 配置证书秘钥
    ssl_certificate_key  2_www.imoocdsp.com.key;

    # ssl会话cache
    ssl_session_cache    shared:SSL:1m;
    # ssl会话超时时间
    ssl_session_timeout  5m;

    # 配置加密套件，写法遵循 openssl 标准
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    
    location / {
        proxy_pass http://tomcats/;
        index  index.html index.htm;
    }
 }                                                        

```

4. reload nginx
./nginx -s reload
