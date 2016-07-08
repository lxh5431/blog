---
title: jvm知识总结（二）
date: 2016-06-28 23:54:25
tags: 编程
categories:
tag:
---
#### 垃圾回收机制
如何判断垃圾回收对象
1.引用计数算法
2.根搜索算法
基本原理：**GCRoot对象作为起始点（根）。如果从根到某个对象是可达的，则该对象称为可达对象（存活对象，不可回收对象）。否则就是不可达对象，可以被回收。**
新生代的垃圾收集器通常使用复制算法，将没有被引用的对象清理掉然后即将存活的对象放入老生代
![复制算法](http://o94r16s1l.bkt.clouddn.com/%E5%A4%8D%E5%88%B6%E7%AE%97%E6%B3%95)
## 触发条件：新生代采用“空闲指针”的方式来控制GC触发，指针保持最后一个在新生代分配的对象位置，当有新的对象要分配内存时，用于检查空间是否足够，不够就触发GC。新生代的GC通常叫做young GC，有时候也叫minor GC。
老生代与新生代不同，对象存活的时间比较长，比较稳定，因此采用标记/整理（也叫标记-紧凑，Mark-Compact）算法。
<!--more-->
## 老生代的GC，通常叫做full GC，也叫major GC。老生代有多情况会触发GC，不过一般来说发生频率不高：
1.旧生代空间不足
2.PemanetGeneration空间不足
3.GC晋升老生代的平均大小大于老生代剩余空间大小
4.手动调用system.gc()
##垃圾回收方式：
串行垃圾回收器（Serial Garbage Collector）
并行垃圾回收器（Parallel Garbage Collector）
并发标记扫描垃圾回收器（CMS Garbage Collector）
G1垃圾回收器（G1 Garbage Collector）
## 优化的复制算法
![优化的复制算法](http://o94r16s1l.bkt.clouddn.com/%E4%BC%98%E5%8C%96%E7%9A%84%E5%A4%8D%E5%88%B6%E7%AE%97%E6%B3%95.png)
解释：
Eden+S0可分配新生对象；
对Eden+S0进行垃圾收集，存活对象复制到S1。清理Eden+S0。一次新生代GC结束。
Eden+S1可分配新生对象；
对Eden+S1进行垃圾收集，存活对象复制到S0。清理Eden+S1。二次新生代GC结束。
循环1。
fullGC、minorGC、magorGC还有youngGC是Java垃圾处理机制（GC）的名词，区分这几个概念非常简单：

老生代进行一次垃圾清理，被称为fullGC或者magorGC。

新生代进行一次垃圾清理，被称为youngGC或者minorGC。
原则：
降低youngGC的频率、减少fullGC的次数。
java内存的优化主要是通过合理的控制GC来实现，主要原则：
1. 不能只看操作系统级别Java进程所占用的内存，这个数值不能准确的反应堆内存的真实占用情况（因为GC过后这个值是不会变化的）。
2. 使用JDK提供的内存查看工具，比如JConsole和Java VisualVM。
3. 优化内存主要的目的是降低youngGC的频率、减少fullGC的次数 ，过多的youngGC和fullGC是会占用很多的系统资源（主要是CPU），影响系统的吞吐量。
