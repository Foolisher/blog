---
title: gc
p: jvm/gc
date: 2018-04-04 15:33:21
tags:
  - jvm
  - gc
---

## 内存模型

### 堆区分代

![](http://fuxiao.oss-cn-shanghai.aliyuncs.com/blog/072251.png)



## GC算法

#### 串行回收(SerialNew)

它是一个新生代串行回收算法

#### 串行并发回收(ParSerialNew)

是一个新生代并发的串行回收算法

#### ParSerialScanvge

#### CMS(ConcurrentMarkSweep)并发标记清除

#### G1



### 分代回收

- 分代占比
- 分代类型

### 引用计数

- 无法解决死循环

### 复制清除

### 根引用法

此法可以避免循环引用导致无法回收的问题

理由是一个对象如果无法达到最初的Root，说明Root也已回收，那Root所使用的对象也可回收了



## GC调优

##### FullGC频繁

