---
title: nginx跨域
date: 2021-02-12 08:37:16
tags: nginx
---

###
   1 首先什么是跨域
```xml
    跨域是指跨域名的访问，分为三种情况:
       1 域名不同的跨域
       2 域名相同，端口不同的跨域
       3 二级域名不同的跨域。

```

### 2 那为什么有跨域问题？
```xml
    假设我有一个网站，另一个人看我不爽，天天通过不同的渠道疯狂攻击我的网站。
      
    如果没有跨域问题，他就可以通过跨域不断地对我的网站发送ajax请求从而攻击。
      
    所以跨域问题本质上是浏览器对于ajax请求的一种安全限制，这本身并不是坏事，更安全。
```
      
  
      
### 3 如何解决跨域问题 (CORS 跨域资源共享)     
####  1 Nginx 解决跨域问题
> 在nginx的配置文件中加入一下配置内容就可以
```xml
    # 允许跨域请求的域， *代表所有
       add_header 'Access-Control-Allowe-Origin' *;
       # 允许带上cookie的请求
       add_header 'Access-Control-Allowe-Crendentials' 'true';
       # 允许请求的方法， 比如GET/POST/PUT/DELETE
       add_header 'Access-Control-Allowe-Methods' *;
       # 允许请求的header
       add_header 'Access-Control-Allowe-Headers' *; 
```
   
####  2 Jsonp 解决跨域问题
####  3 SpringBoot Cors 解决跨域问题
> java后段进行解决跨域问题

```java
    
   @Slf4j
       public class CorsFilter implements Filter {
           @Override
           public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
               HttpServletRequest request = (HttpServletRequest) req;
               HttpServletResponse response = (HttpServletResponse) res;
               response.setHeader("Access-Control-Allow-Origin", "*");
               response.setHeader("Access-Control-Allow-Credentials", "true");
               response.setHeader("Access-Control-Allow-Methods", "POST,GET,OPTIONS,PUT,DELETE,PATCH,HEAD");
               response.setHeader("Access-Control-Max-Age", "3600");
               response.setHeader("Access-Control-Allow-Headers", "X-Requested-With, Content-Type, Authorization, Origin, Accept, Access-Control-Request-Method, Access-Control-Request-Headers");
       
               if("OPTIONS".equalsIgnoreCase(request.getMethod())){
                   response.setStatus(HttpServletResponse.SC_OK);
               }else{
                   chain.doFilter(req, res);
               }
           }
           @Override
           public void init(FilterConfig filterConfig) throws ServletException
           {
       
           }
           @Override
           public void destroy() {}
       
       }
 

```

## 4 Keepalived 提高吞吐量
```xml

keepalived： 设置长连接处理的数量
proxy_http_version：设置长连接http版本为1.1
proxy_set_header：清除connection header 信息

upstream tomcats {
#       server 192.168.1.173:8080 max_fails=2 fail_timeout=1s;
        server 192.168.1.190:8080;
#       server 192.168.1.174:8080 weight=1;
#       server 192.168.1.175:8080 weight=1;
        keepalive 32;
}

server {
        listen       80;
        server_name  www.tomcats.com;

        location / {
            proxy_pass  http://tomcats;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
        }
    }

```

      
