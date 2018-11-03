---
layout: post
title: "Zookeeper集群搭建"
date: 2017-04-13 21:25:29 +0800
comments: true
categories: Dev
tags: [Zookeeper]
---

<!--more-->

## 环境安装

### java环境

```sh
yum install java-1.8.0-openjdk-headless.x86_64
```

### 下载zookeeper安装包

```sh
wget http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
```
### 配置
```sh
tar zxvf zookeeper-3.4.9.tar.gz
cd zookeeper-3.4.9/conf
# 修改zoo.cfg
cp zoo_sample.cfg zoo.cfg
```

* 配置文件中主要修改

```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.

# 修改zk数据文件存储位置
dataDir=/root/zookeeper
# the port at which the clients will connect

# 修改zk 客户端连接端口
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

# zk集群的端口
server.0=192.168.138.128:20881:30881
server.1=192.168.138.129:20881:30881
server.2=192.168.138.130:20881:30881
```

        在对应的机器上将server.x=xxx.xxx.xx.xx:20881:30881改为0.0.0.0:20881:30881

* 参数的含义

> server.0、server.1、server.2 是指整个zookeeper集群内的节点列表。server的配置规则为：  server.N=YYY:A:B  
N表示服务器编号  
YYY表示服务器的IP地址  
A为LF通信端口，表示该服务器与集群中的leader交换的信息的端口。  
B为选举端口，表示选举新leader时服务器间相互通信的端口（当leader挂掉时，其余服务器会相互通信，选择出新的leader）  
一般来说，集群中每个服务器的A端口都是一样，每个服务器的B端口也是一样。但是当所采用的为伪集群时，IP地址都一样，只能是A端口和B端口不一样。  

### 启动前配置
    在zk数据dataDir存储目录/root/zookeeper下新建myid文件，并分别写入0,1,2，与配置中的相同

启动服务

```
bin/zkServer.sh start
```

### 查看服务状态
```sh
# zookeeper1
[root@master zookeeper-3.4.9]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /root/zookeeper-3.4.9/bin/../conf/zoo.cfg
Mode: follower

# zookeeper2
root@ubuntu:~/zookeeper-3.4.9# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /root/zookeeper-3.4.9/bin/../conf/zoo.cfg
Mode: leader
```

### 遇到的问题

```
2017-04-11 20:48:09,120 [myid:2] - WARN  [/0.0.0.0:30881:QuorumCnxManager@400] - Cannot open channel to 0 at election address /192.168.138.128:30881
java.net.NoRouteToHostException: No route to host (Host unreachable)
	at java.net.PlainSocketImpl.socketConnect(Native Method)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
	at java.net.Socket.connect(Socket.java:589)
	at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectOne(QuorumCnxManager.java:381)
	at org.apache.zookeeper.server.quorum.QuorumCnxManager.receiveConnection(QuorumCnxManager.java:295)
	at org.apache.zookeeper.server.quorum.QuorumCnxManager$Listener.run(QuorumCnxManager.java:543)
2017-04-11 20:48:09,121 [myid:2] - INFO  [/0.0.0.0:30881:QuorumPeer$QuorumServer@149] - Resolved hostname: 192.168.138.128 to address: /192.168.138.128
```

这个出现在了一台centos机器上，主要原因是防火墙禁止访问
最简单的方法是禁止防火墙

```sh
systemctl stop firewalld.service
systemctl disable firewalld.service #禁止firewall开机启动
```

另外一种开启端口访问

```sh
/sbin/iptables -I INPUT -p tcp --dport 2181 -j ACCEPT
/sbin/iptables -I INPUT -p tcp --dport 20881 -j ACCEPT
/sbin/iptables -I INPUT -p tcp --dport 30881 -j ACCEPT
```
* 同步服务器时间

```bash
yum install nptdate

ntpdate time.nist.gov
另外的时间服务器
time.nist.gov
time.nuri.net
0.asia.pool.ntp.org
1.asia.pool.ntp.org
2.asia.pool.ntp.org
3.asia.pool.ntp.org
```
