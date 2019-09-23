---
title: JVM参数设置及调优工具
typora-root-url: ../../source/
date: 2019-09-16 19:09:26
tags:
- java
- 面试
- jvm
---

jvm辣么多参数，记起来太困难了，偷偷给自己总结一下，按需查询～

# JVM参数设置

## 内存区域

- Java堆
```bash
-Xmx #最大堆容量
-Xms #最小堆容量
-XX:+/-UseTLAB #是否使用Thread Local Allocation Buffer(TLAB)
-XX:+HeapDumpOnOutOfMemoryError #内存异常时dump出当前的内存堆转储快照
```
<!--more-->

- 方法区
```bash
-XX:PermSize
-XX:MaxPermSize #方法区大小
```
- 虚拟机栈
```bash
-Xss #设置栈内存容量
```
- 本地方法栈
```bash
-Xoss #设置本地方法栈大小（在hotspot中无效，因为不区分虚拟机栈和本地方法栈）
```
- 直接内存
```bash
-XX:MaxDirectMemorySize #设置最大直接内存大小
```

## 垃圾回收

```bash
-XX:+PrintGCDetails #打印内存回收日志
```

- 新生代

```bash
-Xmn #新生代大小
-XX:SurvivorRatio #Eden与Survivor区的比例
-XX:PretenureSizeThreshold #晋升老年代的对象大小
-XX:MaxTenuringThreshold #晋升到老年代的对象年龄
```

- 方法区回收（永久代）

```bash
-Xnoclassgc #是否对类进行回收
-verbose:class
-XX:+TraceClassLoading
-XX:+TraceClassUnLoading #查看类加载和卸载信息
```

- Serial

```bash
-XX:+UseSerialGC #使用Serial + SerialOld
-XX:SurvivorRatio
-XX:PretenureSizeThreshold
-XX:HandlePromotionFailure
```

- ParNew

```bash
-XX:+UseParNewGC #使用ParNew + SerialOld
-XX:ParallelGCThreads #限制垃圾收集的线程数
```

- Parallel Scavenge

```bash
#ParNew中的所有参数，以及如下参数：

-XX:+UseParallelGC #使用ParallelScavenge + SerialOld
-XX:MaxGCPauseMillis #垃圾回收最大停顿时间
-XX:GCTimeRatio #设置吞吐量大小
-XX:+UseAdaptiveSizePolicy #自适应，使用后无需指定新生代大小，比例等参数
```

- ParallelOld

```bash
-XX:+UseParallelOldGC #使用ParallelScavenge + ParallelOld
```

- CMS(Concurrent Mark Sweep)

```bash
-XX:+UseConcMarkSweepGC #使用ParNew + CMS + SerialOld
-XX:CMSInitiatingOccupancyFraction #触发老年代GC的内存占用比例
-XX:HandlePromotionFailure #是否允许担保失败
-XX:+UseCMSCompactAtFullCollection #FullGC时开启内存碎片的合并整理过程（默认开启）
-XX:CMSFullGCsBeforeCompaction #执行多少次FullGC后进行内存压缩（默认为0，即每次都会压缩）
```

# JVM工具

## 监控和故障处理工具

| 名称    | 主要作用                                                     |
| ------- | ------------------------------------------------------------ |
| jps     | JVM Process Status Tool 显示指定系统内所有的HotSpot虚拟机进程 |
| jstat   | JVM Staticstics Monitoring Tool 用于收集虚拟机各方面的运行数据 |
| jinfo   | Configuration Info for Java 显示虚拟机配置信息               |
| jmap    | Memory Map for Java 生成虚拟机内存转储快照(heapdump文件)     |
| jhat    | JVM Heap Dump Browser 用于分析heapdump文件,它会建立一个http/html服务器,让用户可以在浏览器上查看分析结果 |
| jstatck | Stack Trace for Java 显示虚拟机的线程快照                    |

### jps

```bash
jps [options] [hostid]
```

它的功能和unix命令ps类似: 可以列出正在运行的虚拟机进程, 并显示虚拟机执行主类名称及这些进程的本地虚拟机唯一ID(Local Virtual Machine Identifier , LVMID). 

| 选项 | 作用                                             |
| ---- | ------------------------------------------------ |
| -q   | 只输出LVMID,省略主类名                           |
| -m   | 输出虚拟机进程启动时传递给主类的main()函数的参数 |
| -l   | 输出主类的全名,如果进程执行的是jar包,输出jar路径 |
| -v   | 输出虚拟机进程启动时JVM参数                      |

待补充https://www.jianshu.com/p/6357eb854d08                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              

