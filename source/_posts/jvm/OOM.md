---
title: OOM
date: 2022-01-19 19:02:16
tags: jvm
---
# java OOM的情况以及如何排查

## 1 OOM如何产生的
* 1 分配过少：JVM 初始化内存小，业务使用了大量内存；或者不同 JVM 区域分配内存不合理
* 2 代码漏洞：某一个对象被频繁申请，不用了之后却没有被释放，导致内存耗尽
> 内存泄漏: 申请使用完的内存没有释放，导致虚拟机不能再次使用这个内存，这段内存就泄漏了。
因为申请者不用了，而又不能被虚拟机分配给别人用.
> 申请的内存超出了JVM能提供的内存大小

## 2 常见的几种OOM异常

### 2.1 java.lang.OutOfMemoryError: PermGen space(jdk 1.7) 永久代异常, Metadata space(jdk 1.8)原空间异常
```xml
ava7 永久代（方法区）溢出，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。每当一个类初次加载的时候，元数据都会存放到永久代
一般出现于大量 Class 对象或者 JSP 页面，或者采用 CgLib 动态代理技术导致
我们可以通过 -XX：PermSize 和 -XX：MaxPermSize 修改方法区大小
```

### 2.2 java.lang.StackOverflowError 栈内存异常异常
```xml
虚拟机栈溢出，是由于程序中存在死循环或者深度调度造成的，也可能是栈的大小设置的有问题，可以通过-Xss设置大小
```


### 2.3 java.lang.OutOfMemoryError: Java heap space

```xml
堆内存溢出，原因一般由于 JVM 堆内存设置不合理或者内存泄漏导致
如果是内存泄漏，可以通过工具查看泄漏对象到 GC Roots 的引用链。
掌握了泄漏对象的类型信息以及 GC Roots 引用链信息，就可以精准地定位出泄漏代码的位置
如果不存在内存泄漏，就是内存中的对象确实都还必须存活着，
那就应该检查虚拟机的堆参数（-Xmx 与 -Xms），查看是否可以将虚拟机的内存调大些
```

## 3 查看 JVM 内存分布
输入命令查看jvm的内存分布 jmap -heap 15672
```xml
[xxx@xxx ~]# jmap -heap 15162
Attaching to process ID 15162, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.161-b12
​
using thread-local object allocation.
Mark Sweep Compact GC
​
Heap Configuration:
   MinHeapFreeRatio         = 40 # 最小堆使用比例
   MaxHeapFreeRatio         = 70 # 最大堆可用比例
   MaxHeapSize              = 482344960 (460.0MB) # 最大堆空间大小
   NewSize                  = 10485760 (10.0MB) # 新生代分配大小
   MaxNewSize               = 160759808 (153.3125MB) # 最大新生代可分配大小
   OldSize                  = 20971520 (20.0MB) # 老年代大小
   NewRatio                 = 2 # 新生代比例
   SurvivorRatio            = 8 # 新生代与 Survivor 比例
   MetaspaceSize            = 21807104 (20.796875MB) # 元空间大小
   CompressedClassSpaceSize = 1073741824 (1024.0MB) # Compressed Class Space 空间大小限制
   MaxMetaspaceSize         = 17592186044415 MB # 最大元空间大小
   G1HeapRegionSize         = 0 (0.0MB) # G1 单个 Region 大小
​
Heap Usage:  # 堆使用情况
New Generation (Eden + 1 Survivor Space): # 新生代
   capacity = 9502720 (9.0625MB) # 新生代总容量
   used     = 4995320 (4.763908386230469MB) # 新生代已使用
   free     = 4507400 (4.298591613769531MB) # 新生代剩余容量
   52.56726495150862% used # 新生代使用占比
Eden Space:  
   capacity = 8454144 (8.0625MB) # Eden 区总容量
   used     = 4029752 (3.8430709838867188MB) # Eden 区已使用
   free     = 4424392 (4.219429016113281MB) # Eden 区剩余容量
   47.665996699370154% used  # Eden 区使用占比
From Space: # 其中一个 Survivor 区的内存分布
   capacity = 1048576 (1.0MB)
   used     = 965568 (0.92083740234375MB)
   free     = 83008 (0.07916259765625MB)
   92.083740234375% used
To Space: # 另一个 Survivor 区的内存分布
   capacity = 1048576 (1.0MB)
   used     = 0 (0.0MB)
   free     = 1048576 (1.0MB)
   0.0% used
tenured generation: # 老年代
   capacity = 20971520 (20.0MB)
   used     = 10611384 (10.119804382324219MB)
   free     = 10360136 (9.880195617675781MB)
   50.599021911621094% used
​
10730 interned Strings occupying 906232 bytes.
```
另外，可以在 JVM 运行时查看最耗费资源的对象，jmap -histo:live 15162 | more
> jmap -histo:live 执行此命令，JVM 会先触发 GC，再统计信息

## 4 dump文件分析

Dump 文件是 Java 进程的内存镜像，其中主要包括 系统信息、虚拟机属性、完整的线程 Dump、所有类和对象的状态 等信息
当程序发生内存溢出或 GC 异常情况时，怀疑 JVM 发生了 内存泄漏，这时我们就可以导出 Dump 文件分析
JVM 启动参数配置添加以下参数
```xml
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=./（参数为 Dump 文件生成路径）

```
当 JVM 发生 OOM 异常自动导出 Dump 文件，文件名称默认格式：java_pid{pid}.hprof

上面配置是在应用抛出 OOM 后自动导出 Dump，或者可以在 JVM 运行时导出 Dump 文件
```xml
  jmap -dump:file=[文件路径] [pid]
  jmap -dump:file=./jvmdump.hprof 15162
```




