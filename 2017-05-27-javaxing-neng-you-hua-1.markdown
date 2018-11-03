---
layout: post
title: "Java性能优化一线程DUMP"
date: 2017-05-27 09:43:06 +0800
comments: true
categories: Dev
tags: [Java, 性能优化]
---

参考: [三个实例演示 Java Thread Dump 日志分析](http://www.cnblogs.com/zhengyun_ustc/archive/2013/01/06/dumpanalysis.html)

<!-- more -->
### Java程序线程Thread Dump分析

通过`jstack -l PID`可以实时查看当前程序的线程dump

```
"corgi-consumer-workers-ad.unionbusiness.unionalarm-7-1" #188 prio=5 os_prio=0 tid=0x00007f9e6006b000 nid=0x6297 waiting for monitor entry [0x00007f9dd2cb5000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.mogujie.union.adexchange.biz.process.DefaultMessageHandler.assembleAdResult(DefaultMessageHandler.java:116)
        - waiting to lock <0x00000006c3bb2440> (a java.lang.Object)
        at com.mogujie.union.adexchange.biz.process.DefaultMessageHandler.handle(DefaultMessageHandler.java:79)
        at com.mogujie.union.adexchange.common.corgi.CorgiConsumeInitializer.lambda$newListener$0(CorgiConsumeInitializer.java:32)
        at com.mogujie.union.adexchange.common.corgi.CorgiConsumeInitializer$$Lambda$9/799107722.onReceived(Unknown Source)
        at com.mogujie.corgi.client.consumer.consume.AbstractParallelWorkerGroup.consume0(AbstractParallelWorkerGroup.java:92)
        at com.mogujie.corgi.client.consumer.consume.AbstractParallelWorkerGroup.consume(AbstractParallelWorkerGroup.java:145)
        at com.mogujie.corgi.client.consumer.consume.SingleParallelWorkerGroup$1.run(SingleParallelWorkerGroup.java:73)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at io.netty.util.concurrent.DefaultThreadFactory$DefaultRunnableDecorator.run(DefaultThreadFactory.java:144)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - <0x00000006c3b92898> (a java.util.concurrent.ThreadPoolExecutor$Worker)
```

线程状态有
`jstack  -l PID` 查看java程序线程情况  
•  死锁，Deadlock（重点关注）   
•  等待资源，Waiting on condition（重点关注）   
•  等待获取监视器，Waiting on monitor entry（重点关注）   
•  阻塞，Blocked（重点关注）   
•  执行中，Runnable   
•  暂停，Suspended   
•  对象等待中，Object.wait() 或 TIMED_WAITING   
•  停止，Parked   

* 我们可以看到线程处于`Blocked`阻塞状态。说明线程等待资源超时！
* `waiting to lock <0x00000006c3bb2440> `指线程等待给`0x00000006c3bb2440`这个地址上锁。在线程dump文件中我们可以搜索到很多线程在等待给这个地址上锁，所以找到谁获得了这个锁，就可以找到问题的根源。
* `waiting for monitor entry [0x00007f9dd2cb5000]` 指线程通过同步块`synchronized(obj){...}`申请进入了临界区，从而进入了等待的`Entry Set`队列。
* 第一行中 `corgi-consumer-workers-ad.unionbusiness.unionalarm-7-1`是线程名(Thread Name); `prio`是线程优先级，`tid`是`Java Thread id`, `nid`是`native`线程`id`。 

#### 线DUMP中的注意事项
1、不同的Java虚拟机的线程dump的内容是不一样的，JVM版本版本不同，dump信息也略有差异。  
2、在实际运行中，一次dump可能只表示当时时刻的情况，还无法确认问题，所以建议多次dump信息，如果多次的结果指向同一个地方，则可以确定问题的原因。  
3、线程状态中，Deadlock: 死锁线程，一般指多个线程调用，进入相互资源调用，导致一直等待无法释放的情况。  
4、`waiting on condition`: 等待资源，或等待某个条件的发生。  
5、`waiting for monitor entry`和 `Object.wait()`: `Monitor`是 Java中用以实现线程之间的互斥与协作的主要手段，它可以看成是对象或者 Class的锁。每一个对象都有，也仅有一个 monitor。
从下图1中可以看出，每个 `Monitor`在某个时刻，只能被一个线程拥有，该线程就是 "Active Thread"，而其它线程都是 "Waiting Thread"，分别在两个队列 “ Entry Set”和 “Wait Set”里面等候。在 “Entry Set”中等待的线程状态是 "Waiting for monitor entry"，而在 "Wait Set"中等待的线程状态是 "in Object.wait()"。

![http://zhangwenqiang.com.cn/java_monitor.png](http://zhangwenqiang.com.cn/java_monitor.png)


#### CPU飙高问题排查
* 通过top命令查看CPU占用较高的进程ID
* 通过jstack命令将java的线程栈输出，保留现场`jstack -l 30142 > 30142.stack`
* 通过`top -H -p PID`命令输出占用cpu过高的线程 找到占用cpu过高的PID
* 使用printf 命令将30450转换成16进制。`printf "%x\n" 18430`
* 打开之前保存的stack文件，找到线程地址为`0x76f2`的输出，即为出问题的线程









