---
layout: post
title: "堆内存最佳实践"
date: 2017-06-08 01:05:38 +0800
comments: true
categories: Dev
tags: [Java, 性能优化]
---

本文主要介绍内存分析的几种方法

<!-- more -->

## 堆分析

### 采用堆直方图分析堆内存使用
利用堆直方图，我们可以快速看到应用内的对象数目，同时不需要进行完整的堆转储。堆直方图获取方式`jcmd PID GC.class_histogram`

```sh
$ jcmd 16685 GC.class_histogram | more
16685:

 num     #instances         #bytes  class name
----------------------------------------------
   1:       3382715      310396224  [C
   2:       3293714       79049136  java.lang.String
   3:       2364000       75648000  java.util.HashMap$Node
   4:         14080       54832544  [I
   5:         95607       23472416  [B
   6:        159142       22423488  [Ljava.util.HashMap$Node;
   7:         53314       13002496  [Ljava.lang.Object;
   8:        524288       12582912  com.mogujie.union.cpmbilling.common.disruptor.CpmLogEvent
   9:        472423       11338152  java.util.concurrent.atomic.AtomicLong
  10:        164987        7919376  java.util.HashMap
  11:        142903        4572896  com.mogujie.union.cpmbilling.common.kafka.consumer.MessageInfo
  12:        161610        3878640  java.lang.Long
  13:        131572        3157728  com.mogujie.union.cpmbilling.common.kafka.producer.KafkaLogEntity
  14:        169888        2718208  java.lang.Integer
......
```

在接近顶端的地方，字符数组(`[C`)和`String`对象很常见，因为他们是最常创建的Java对象。字节数组(`[B`)和Object数组同样很常见。该命令不会强制执行Full GC，但是`GC.class_histogram`中的输出仅包含活跃对象。

运行下面的命令也会得到同样的结果
```sh
jmap -histo PID
```
`jmap`的输出中包含会被回收的对象(死对象)。如果在获取直方图之前强制执行一次Full GC，可以使用下面命令
```sh
jmap -histo:live PID
```
直方图擅长识别有分配了一两个特定类的过多实例而引发的问题，但是要进行深度的分析，就需要应用到堆转储。

### 堆转储

生成堆转储文件的方式

```sh
jcmd PID GC.heap_dump /path/to/save/heap_dump.hprof
```
或者
```sh
jmap -dump:live,file=/path/to/save/heap_dump.hprof PID
```
在jmap中包含live选项，这会在生成堆转储之前强制执行一次Full GC; jcmd默认就会这样做。如果你想包含其他对象(死对象)，可以在jcmd命令后面加上-all选项。

生成的heap_dump.hprof文件我们可以使用下面一些工具打开，常见的有:  
`jhat`: 这是最原始的堆分析工具，它会读取堆转储文件，并运行一个小型的http服务器，可以在网页上查看堆转储的信息。

VisualVM 以及 mat等开源原件

对堆的第一遍分析通常涉及保留内存。一个对象的保留内存，是指回收该对象可以释放出的内存量。

#### 浅对象大小、保留对象大小及深对象大小
>
对于内存分析，还有其他两个很有用的术语: 浅(shallow)和深(deep)。一个对象的浅大小，值的是该对象本身的大小。如果该对象包含一个指向另一个对象的引用，4字节或8字节的引用会计算在内，但是目标对象的大小不会包含进来。

> 深大小则包含哪些对象的大小。深大小与保留大小的区别在于那些存在共享的对象。在深大小包括那些共享的内存空间，而保留大小则不包含。

### 内存溢出错误

* JVM没有原生内存可用
* 永久代(在Java 7和更早的版本中) 或元空间(在Java 8)内存不足。
* Java堆本身空间不足 - 对于给定的堆空间而言，应用中活跃对象太多
* JVM执行GC耗时太多

自动堆转储
OutOfMemoryError是不可预料的，我们很难确定应该何时获得堆转储。有几个JVM参数可以帮助我们获取

-XX:+HeapDumpOnOutOfMemoryError
该标志默认为false，打开该标志，JVM会在抛出OutOfMemoryError时创建堆转储。

-XX:HeapDumpPath=<path>
该标志执行堆转储存储路径。默认会生成java_pid.hprof文件

-XX:+HeapDumpAfterFullGC
这会在运行一次Full GC后生成一个堆转储文件

-XX:+HeapDumpBeforeFullGC
这回在运行一次Full GC之前生成一个堆转储文件


### 减少内存使用

#### 减少对象大小
减少对象大小的大小有2种方式：减少实例变量的个数(效果很明显),或者减小实例变量的大小。
