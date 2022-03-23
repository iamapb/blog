---
title: UDP time_wait和close_wait 产生过多原因和解决犯法
date: 2022-02-18 09:44:08
tags: 网络
---
## 问题：
> 有时会碰到服务器有大量的socket处于CLOSE_WAIT状态，也无法关闭，导致服务器无法接受新的用户请求，最终导致服务器奔溃，系统重启才能解决

### 2.tcp中止的四次挥手

由于tcp连接是全双工的，因此每个方向都必须是单独进行关闭的。
![四次挥手](/../../static/网络/tcp四次挥手.jpg)

当clien端传输完成数据，或者需要断开连接时：

```xml
Client端发送一个FIN报文给Server端。(序号为M)
  1.1. 表示要终止Client到Server这个方向的连接。
  1.2. 通过调用close(socket) API。
  1.3 表示Client不再会发送数据到Server端。(但Server还能继续发给Client端)
  1.4 Client状态变为FIN_WAIT_1 Server端收到FIN后，发送一个ACK报文给Client端。(序号为M+1)

  2.1 Server状态变为CLOSE_WAIT
  2.2 Client收到序号为(M+1)的ACK后状态变为FIN_WAIT_2


Server端也发送一个FIN报文给Client端。(序号为N)
3.1 表示Server也要终止到Client端这个方向的连接。
3.2. 通过调用close(socket) API。
3.3 Server端状态变为LAST_ACK
Client端收到报文FIN后，也发送一个ACK报文给服务器。(序号N+1)
4.1 Client状态变为TIME_WAIT
Server端收到序号为(N+1)的ACK
5.1 Server的状态变为CLOSED.
等带2MSL之后
6.1 Client的状态也变为CLOSE.
至此，一个完整的TCP连接就关闭了。
两个基本问题：

Q: 我们看到CLOSE_WAIT出现在什么时候呢？
A: 在Sever端收到Client的FIN消息之后。
Q: 状态CLOSE_WAIT在什么时候转换成下一个状态呢？
A: 在Server端向Client发送FIN消息之后。
至此似乎明白了为什么会出现CLOSE_WAIT的状态：如果Server端一直没有向client端发送FIN消息(调用close() API)，那么这个CLOSE_WAIT会一直存在下去。
```

### 2.原因分析

```xml
  从上面我们看到出现CLOSE_WAIT，说明Server端没有发起close()操作，这基本上是用户server端程序的问题了；通常情况下，Server都是等待Client访问，如果Client退出请求关闭连接，server端自觉close()对应的连接。

 当然这也可能是业务实现上的需要，暂时不发送FIN，因为服务器可能还有数据要发往客户端，等发送完所有应用数据最后再发送FIN消息了；这个场景并不是这里我们讨论的大量COLSE_WAIT的问题了，因为这个还是可控的。

```

### 3 其他场景

我们知道一个进程打开一个socket，然后此进程fork出子进程的时候，父进程已打开的socket是会被继承的，即子进程能够继续访问这个socket。其结果就是，一个socket被两个进程打开，一个父进程和一个子进程，此时socket的引用计数会变成2。
```xml
  1.调用close(socket)时，内核先检查socket上的引用计数器：如果引用计数大于1，那么将这个引用计数减1，然后直接返回。如果引用计数等于1，那么内核才会真正关闭此socket。(通过发送FIN到对端来关闭TCP连接)

  2.调用shutdown(socket，HOW)时，内核不会检查此socket对应的引用计数器，直接向对端发送FIN来关闭TCP连接。


据此分析，很大可能性是用户服务器的程序实现有问题导致的大量CLOSE_WAIT的socket，比如父进程打开了socket，然后通过fork出子进程来处理业务，父进程继续对网络请求进行监听，永远不会终止；当客户端发FIN过来的时候，处理业务的子进程处理此FIN消息，调用close()对本端进行关闭，然而这个close()调用只是把socket的引用计数器减1，因为父进程还在运行，socket并没关闭，这样就导致系统中又多了一个CLOSE_WAIT的socket，长此以往，就这样了。
```

关于TIME_WAIT状态

多说两句关于TIME_WAIT的状态，这个发生在client端，而且是不可避免的，其时间长度是固定的2MSL，到期自动转为CLOSED，不会导致系统资源耗尽的问题。MSL是一个系统级参数，可调。
