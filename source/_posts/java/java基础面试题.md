---
title: java基础面试题
date: 2021-02-24 10:55:09
tags: java
---
### 1  字符型常量和字符串常量的区别?
1.形式上: 字符型常量是单引号引起的一个字符; 字符串常量是双引号引起的 0 个或若干个字符

2.含义上: 字符型常量相当于一个整型值( ASCII 值),可以参加表达式运算; 字符串常量代表一个地址值(该字符串在内存中存放位置)

3.占内存大小 字符常量只占 2 个字节; 字符串常量占若干个字节 (注意： char 在 Java 中占两个字节),

> 字符封装类 Character 有一个成员常量 Character.SIZE 值为 16,单位是bits,该值除以 8(1byte=8bits)后就可以得到 2 个字节

### 2 java 泛型了解？什么是类型擦除？介绍一下常用的通配符？
1 Java 泛型（generics）是 JDK 5 中引入的一个新特性, 泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。

```java
    List<Integer> list = new ArrayList<>();
    
    list.add(12);
    //这里直接添加会报错
    list.add("a");
    Class<? extends List> clazz = list.getClass();
    Method add = clazz.getDeclaredMethod("add", Object.class);
    //但是通过反射添加，是可以的
    add.invoke(list, "kl");
    
    System.out.println(list)

```

2 类型擦除 简单的理解就是在java的编译期间，所有的范型信息都会被擦掉，原始类型就为object [详情了解请看](https://www.cnblogs.com/wuqinglong/p/9456193.html)

3 常用的通配符为： T，E，K，V，？
  
 > ？ 表示不确定的 java 类型
  T (type) 表示具体的一个 java 类型
  K V (key value) 分别代表 java 键值中的 Key Value
  E (element) 代表 Element

### 3 == 和 equals的区别
== : 它的作用是判断两个对象的地址是不是相等。即判断两个对象是不是同一个对象。(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)

因为 Java 只有值传递，所以，对于 == 来说，不管是比较基本数据类型，还是引用数据类型的变量，其本质比较的都是值，只是引用类型变量存的值是对象的地址。

equals() : 它的作用也是判断两个对象是否相等，它不能用于比较基本数据类型的变量。equals()方法存在于Object类中，而Object类是所有类的直接或间接父类。

Object类equals()方法：

public boolean equals(Object obj) {
     return (this == obj);
}
Copy to clipboardErrorCopied
equals() 方法存在两种使用情况：

+ 情况 1：类没有覆盖 equals()方法。则通过equals()比较该类的两个对象时，等价于通过“==”比较这两个对象。使用的默认是 Object类equals()方法。
+ 情况 2：类覆盖了 equals()方法。一般，我们都覆盖 equals()方法来两个对象的内容相等；若它们的内容相等，则返回 true(即，认为这两个对象相等)。

举个例子：
```java
    public class test1 {
        public static void main(String[] args) {
            String a = new String("ab"); // a 为一个引用
            String b = new String("ab"); // b为另一个引用,对象的内容一样
            String aa = "ab"; // 放在常量池中
            String bb = "ab"; // 从常量池中查找
            if (aa == bb) // true
                System.out.println("aa==bb");
            if (a == b) // false，非同一对象
                System.out.println("a==b");
            if (a.equals(b)) // true
                System.out.println("aEQb");
            if (42 == 42.0) { // true
                System.out.println("true");
            }
        }
    }
    Copy to clipboardErrorCopied
    说明：
    
    String 中的 equals 方法是被重写过的，因为 Object 的 equals 方法是比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。
    当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象。
    
```
String类equals()方法：
```java
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

### 4 1.4.4. 深拷贝 vs 浅拷贝
  浅拷贝：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。
  深拷贝：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。
  
  ![照片](/../../static/拷贝.jpg)
  
### 5 在 Java 中定义一个不做事且没有参数的构造方法的作用
  > Java 程序在执行子类的构造方法之前，如果没有用 super()来调用父类特定的构造方法，则会调用父类中“没有参数的构造方法”。因此，如果父类中只定义了有参数的构造方法，而在子类的构造方法中又没有用 super()来调用父类中特定的构造方法，则编译时将发生错误，因为 Java 程序在父类中找不到没有参数的构造方法可供执行。解决办法是在父类里加上一个不做事且没有参数的构造方法。
        
### 6. BIO,NIO,AIO 有什么区别?
 >  1.BIO (Blocking I/O): 同步阻塞 I/O 模式，数据的读取写入必须阻塞在一个线程内等待其完成。在活动连接数不是特别高（小于单机 1000）的情况下，这种模型是比较不错的，可以让每一个连接专注于自己的 I/O 并且编程模型简单，也不用过多考虑系统的过载、限流等问题。线程池本身就是一个天然的漏斗，可以缓冲一些系统处理不了的连接或请求。但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理模型来应对更高的并发量。
    2.NIO (Non-blocking/New I/O): NIO 是一种同步非阻塞的 I/O 模型，在 Java 1.4 中引入了 NIO 框架，对应 java.nio 包，提供了 Channel , Selector，Buffer 等抽象。NIO 中的 N 可以理解为 Non-blocking，不单纯是 New。它支持面向缓冲的，基于通道的 I/O 操作方法。 NIO 提供了与传统 BIO 模型中的 Socket 和 ServerSocket 相对应的 SocketChannel 和 ServerSocketChannel 两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模式。阻塞模式使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；非阻塞模式正好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞 I/O 来提升开发速率和更好的维护性；对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发
    3.AIO (Asynchronous I/O): AIO 也就是 NIO 2。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步非阻塞的 IO 模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。AIO 是异步 IO 的缩写，虽然 NIO 在网络操作中，提供了非阻塞的方法，但是 NIO 的 IO 行为还是同步的。对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程自行进行 IO 操作，IO 操作本身是同步的。查阅网上相关资料，我发现就目前来说 AIO 的应用还不是很广泛，Netty 之前也尝试使用过 AIO，不过又放弃了。

### 7 面向对象和面向过程的区别
> +1.⾯向过程 ：⾯向过程性能⽐⾯向对象⾼。 因为类调⽤时需要实例化，开销⽐᫾⼤，⽐᫾消
  耗资源，所以当性能是最重要的考量因素的时候，⽐如单⽚机、嵌⼊式开发、Linux/Unix 等
  ⼀般采⽤⾯向过程开发。但是，⾯向过程没有⾯向对象易维护、易复⽤、易扩展。
  +2.⾯向对象 ：⾯向对象易维护、易复⽤、易扩展。 因为⾯向对象有封装、继承、多态性的特
  性，所以可以设计出低耦合的系统，使系统更加灵活、更加易于维护。但是，⾯向对象性能
  ⽐⾯向过程低.          

### 8 java中只有值传递  

## 2 java 集合
### 2.1 说说 list set map三者的区别

+List (对付顺序的好帮⼿)： 存储的元素是有序的、可重复的。
+Set (注重独⼀⽆⼆的性质): 存储的元素是⽆序的、不可重复的。
+Map (⽤ Key 来搜索的专家): 使⽤键值对（kye-value）存储，类似于数学上的函数
y=f(x)，“x”代表 key，"y"代表 value，Key 是⽆序的、不可重复的，value 是⽆序的、可重复
的，每个键最多映射到⼀个值。



