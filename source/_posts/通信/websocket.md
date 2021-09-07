---
title: websocket
date: 2021-03-03 23:05:50
tags: websocket
---
### springBoot + stomp + websocket 实现前后端实时通信 发送消息通知

> 要想简单的实现前后端的实时通信 可以使用websocket实现

## 1 后段实现 springBoot + stomp
> 1 首先springboot 在pom中 添加websocket 的依赖
```java
    <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-websocket</artifactId>
    </dependency>

```

> 2 写个WebsocketConfig 继承 WebSocketMessageBrokerConfigurer
```java

    import org.springframework.context.annotation.Configuration;  
    import org.springframework.messaging.simp.config.MessageBrokerRegistry;  
    import org.springframework.web.socket.config.annotation.AbstractWebSocketMessageBrokerConfigurer;  
    import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;  
    import org.springframework.web.socket.config.annotation.StompEndpointRegistry;  
      
    @Configuration  
    @EnableWebSocketMessageBroker  //注解开启STOMP协议来传输基于代理的消息，此时控制器支持使用@MessageMapping
    public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {  
      
        @Override  
        public void configureMessageBroker(MessageBrokerRegistry config) {  
            config.enableSimpleBroker("/topic","/user");//topic用来广播，user用来实现p2p
        }  
      
        @Override  
        public void registerStompEndpoints(StompEndpointRegistry registry) {  
            registry.addEndpoint("/webServer").setAllowedOrigins("*").withSockJS();  
            registry.addEndpoint("/ueueServer").setAllowedOrigins("*").withSockJS();//注册两个STOMP的endpoint，分别用于广播和点对点
            // setAllowerOrigin("*") 是用来解决跨域问题   
        }  
      
    }

``` 

> 3(1) websocket 添加 setAllowerOrigin("*) 前端进行通信的时候 还是出现跨域问题

![错误现象](/../../static/websocket_cors.png)

```java
    import lombok.extern.slf4j.Slf4j;
    
    import javax.servlet.*;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    
    @Slf4j
    public class CorsFilter implements Filter {
        @Override
        public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
            HttpServletRequest request = (HttpServletRequest) req;
            HttpServletResponse response = (HttpServletResponse) res;
            response.setHeader("Access-Control-Allow-Origin", "*);       
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


  有可能是 response.setHeader("Access-Control-Allow-Origin", "*"); 这句配置出现了问题 不适合使用通配符*
  可以改成 response.setHeader("Access-Control-Allow-Origin", request.getHeader("Origin)); 
```
  
> 3.(2) 同样还是跨域的问题 可能是  registry.addEndpoint("/webServer") 的路径被程序的拦截器拦截 前端收不到响应 就认为是没有跨域
   同样会报这个跨域问题 还是要仔细排查 这个问题困扰了我一整天
> 4 写个简单的controller 去和前端连接
```java
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.messaging.handler.annotation.MessageMapping;
    import org.springframework.messaging.simp.SimpMessagingTemplate;
    import org.springframework.stereotype.Controller;
    
    @Controller
    public class SubController {
        @Autowired
        public SimpMessagingTemplate template;  
          
        
        @MessageMapping("/subscribe")
        public void subscribe(ReceiveMessage rm) {
            for(int i =1;i<=20;i++) {
                //广播使用convertAndSend方法，第一个参数为目的地，和js中订阅的目的地要一致
                template.convertAndSend("/topic/getResponse", rm.getName());
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
    
        }
        
        @MessageMapping("/queue")
        public void queuw(ReceiveMessage rm) {
            System.out.println("进入方法");
            for(int i =1;i<=20;i++) {
                /*广播使用convertAndSendToUser方法，第一个参数为用户id，此时js中的订阅地址为
                "/user/" + 用户Id + "/message",其中"/user"是固定的*/
                template.convertAndSendToUser("zhangsan","/message",rm.getName());
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
    
        }
    }

```

## 2 前端使用vue + sockJs 进行封装
> 1 在vue的package.json 添加配置项依赖
```xml
    "sockjs-client": "^1.5.0",
    "sortablejs": "1.8.4",
    "stompjs": "^2.3.3",
```
> 2 在vue文件中引入
```xml
    import SockJS from "sockjs-client";
    import Stomp from "stompjs";
```

> 3 在methos中写个方法监听
```xml
    getMyPatientPush() {
          
          if (this.$store.state.user.roles.indexOf("doctor") != -1) {
            this.socket = new SockJS(连接地址); // 路径 http 开头
            this.stompClient = Stomp.over(this.socket);
            this.stompClient.onclose = (err) => {
              console.log("连接断开");
              this.getMyPatientPush();
            };
            this.stompClient.connect(
              {},
              (frame) => {
                // 连接
                // console.log(frame);
                this.stompClient.subscribe(
                  `/user/doctor_${this.userId}/notify`, // 订阅的前缀路径
                  (greeting) => {
                    this.pushMessage();
                    // 这里接受后端推送的数据
                    console.log("++++++++++>", greeting);
                    let newPatient = JSON.parse(greeting.body);
                    this.$EventBus.$emit("myPatientPush", newPatient);
                    this.newPatientData.unshift(newPatient);
                    this.open();
                  }
                );
                this.stompClient.subscribe(
                  `/user/doctor_${this.userId}/heart`,
                  (res) => {}
                );
              },
              (err) => {
                console.log(err);
              }
            );
          }
          let index = 1;
        }

```
   
   

