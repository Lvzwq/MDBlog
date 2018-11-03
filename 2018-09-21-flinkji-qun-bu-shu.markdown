---
layout: post
title: "Flink集群HA模式部署"
date: 2018-09-21 11:11:22 +0800
comments: true
categories: Dev
tags: [Flink]
---

本文主要介绍Flink集群HA模式部署

<!--more-->

## 下载安装
从Flink官网下载最新的稳定版本安装包[https://flink.apache.org/downloads.html](https://flink.apache.org/downloads.html)， 下载1.6.0最新版本。有2种版本，一个是`Apache Flink only`，一种是`Flink with Hadoop® 2.8`。我们安装的是后者。



我们这次安装的5台机器，3台机器作为JobManager,另外4台机器TaskManager,机器Ip分别为:
```
10.50.xx.10 (JobManager)
10.50.xx.11 (JobManager/TaskManager)
10.50.xx.12 (JobManager/TaskManager)
10.50.xx.13 (TaskManager)
10.50.xx.14 (TaskManager)
```

服务依赖zookeeper集群的协调机制实现高可用。zookeeper集群地址
```
10.50.xx.101:2181
10.50.xx.102:2181
10.50.xx.103:2181
```

服务依赖Hadoop进行存储JobManager的元数据以及故障恢复的数据，hadoop地址为
```
10.50.xx.200:9000
```

## 配置文件
Flink 集群基本配置，修改配置文件`flink-1.6.0/conf/flink-conf.yaml`

```yaml
# ip地址设置对应机器的ip
jobmanager.rpc.address: 10.50.xx.10

# The RPC port where the JobManager is reachable.

jobmanager.rpc.port: 6123

# The heap size for the JobManager JVM (JobManager JVM能分配的内存)

jobmanager.heap.mb: 1024m

# The heap size for the TaskManager JVM (TaskManager JVM能分配的内存)

taskmanager.heap.mb: 4096m

# The number of task slots that each TaskManager offers. Each slot runs one parallel pipeline. (机器核心数)

taskmanager.numberOfTaskSlots: 4

# Previously this key was named recovery.zookeeper.quorum

high-availability: zookeeper
high-availability.zookeeper.quorum: 10.50.xx.101:2181,10.50.xx.102:2181,10.50.xx.103:2181
high-availability.zookeeper.path.root: /flink

#  Previously this key high-availability.zookeeper.path.namespace
high-availability.cluster-id: /default_ns  

high-availability.zookeeper.storageDir: hdfs://10.50.xx.200:9000/flink

#
The backend that will be used to store operator state checkpoints if
# checkpointing is enabled.
#
# Supported backends are 'jobmanager', 'filesystem', 'rocksdb', or the
# <class-name-of-factory>.
#
state.backend: filesystem

# Directory for checkpoints filesystem, when using any of the default bundled
# state backends.
#
state.checkpoints.dir: hdfs://10.50.xx.200:9000/flink-checkpoints

# Default target directory for savepoints, optional.
#
state.savepoints.dir: hdfs://10.50.xx.200:9000/flink-savepoints

# Flag to enable/disable incremental checkpoints for backends that
# support incremental checkpoints (like the RocksDB state backend).
#
# state.backend.incremental: false
```

### masters 文件配置
```
10.50.xx.10 
10.50.xx.11 
10.50.xx.12 
```

### slaves 文件配置
```
10.50.xx.11
10.50.xx.12 
10.50.xx.13 
10.50.xx.14 
```

## 修改Flink日志打印
修改`flink-1.6.0/conf/log.properties`下
```
log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.file=${log.file}
log4j.appender.file.append=true
log4j.appender.file.MaxFileSize=512MB
log4j.appender.file.MaxBackupIndex=10
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %-5p %-60c %x - %m%n
```


## 配置免密登录

```bash
ssh-keygen -t rsa
cat id_rsa.pub >> authorized_keys

# 目录文件权限设置
chmod 600 authorized_keys
chmod 700 ~/.ssh

# 将公钥(~/.ssh/id_rsa.pub)复制到其他机器的authorized_keys文件中
```

## 注意问题

* 如果`ssh`免密登录的默认端口不是`22`的话，在`bin/start-cluster.sh`加上

```bash
export FLINK_SSH_OPTS="-p 端口号"
```

2、启动Flink集群报错
```
2018-03-25 23:30:35,783 ERROR org.apache.flink.runtime.jobmanager.JobManager                - Failed to run JobManager.
org.apache.hadoop.security.AccessControlException: Permission denied: user=mapp, access=WRITE, inode="/":hadoop:supergroup:drwxr-xr-x
        at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.check(FSPermissionChecker.java:342)
        at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.checkPermission(FSPermissionChecker.java:251)
        at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.checkPermission(FSPermissionChecker.java:189)
        at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:1744)
        at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:1728)
        at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkAncestorAccess(FSDirectory.java:1687)
        at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:1728)
        at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkAncestorAccess(FSDirectory.java:1687)
        at org.apache.hadoop.hdfs.server.namenode.FSDirMkdirOp.mkdirs(FSDirMkdirOp.java:60)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.mkdirs(FSNamesystem.java:2980)
        at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.mkdirs(NameNodeRpcServer.java:1096)
        at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.mkdirs(ClientNamenodeProtocolServerSideTranslatorPB.java:652)
        at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:503)
        at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:989)
        at org.apache.hadoop.ipc.Server$RpcCall.run(Server.java:868)
        at org.apache.hadoop.ipc.Server$RpcCall.run(Server.java:814)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:422)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1886)
        at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2603)

        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
        at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
        at org.apache.hadoop.ipc.RemoteException.instantiateException(RemoteException.java:121)
        at org.apache.hadoop.ipc.RemoteException.unwrapRemoteException(RemoteException.java:88)
        at org.apache.hadoop.hdfs.DFSClient.primitiveMkdir(DFSClient.java:2527)
        at org.apache.hadoop.hdfs.DFSClient.mkdirs(DFSClient.java:2500)
        at org.apache.hadoop.hdfs.DistributedFileSystem$25.doCall(DistributedFileSystem.java:1155)
        at org.apache.hadoop.hdfs.DistributedFileSystem$25.doCall(DistributedFileSystem.java:1152)
        at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
        at org.apache.hadoop.hdfs.DistributedFileSystem.mkdirsInternal(DistributedFileSystem.java:1152)
        at org.apache.hadoop.hdfs.DistributedFileSystem.mkdirs(DistributedFileSystem.java:1144)
        at org.apache.hadoop.fs.FileSystem.mkdirs(FileSystem.java:1913)
        at org.apache.flink.runtime.fs.hdfs.HadoopFileSystem.mkdirs(HadoopFileSystem.java:159)
        at org.apache.flink.runtime.blob.FileSystemBlobStore.<init>(FileSystemBlobStore.java:61)
        at org.apache.flink.runtime.blob.BlobUtils.createFileSystemBlobStore(BlobUtils.java:126)
        at org.apache.flink.runtime.blob.BlobUtils.createBlobStoreFromConfig(BlobUtils.java:92)
        at org.apache.flink.runtime.highavailability.HighAvailabilityServicesUtils.createHighAvailabilityServices(HighAvailabilityServicesUtils.java:103)
        at org.apache.flink.runtime.jobmanager.JobManager$.runJobManager(JobManager.scala:2010)
        at org.apache.flink.runtime.jobmanager.JobManager$$anonfun$2.apply$mcV$sp(JobManager.scala:2115)
        at org.apache.flink.runtime.jobmanager.JobManager$$anonfun$2.apply(JobManager.scala:2093)
        at org.apache.flink.runtime.jobmanager.JobManager$$anonfun$2.apply(JobManager.scala:2093)
        at scala.util.Try$.apply(Try.scala:192)
        at org.apache.flink.runtime.akka.AkkaUtils$.retryOnBindException(AkkaUtils.scala:755)

```

因为`hadoop`是在`hadoop`用户下部署的，指定`hadoop`的用户，在`bin/config.sh`中加入
```bash
export HADOOP_USER_NAME=hadoop
```


## 参考

* [http://wuchong.me/blog/2016/02/26/flink-docs-setup-cluster/](http://wuchong.me/blog/2016/02/26/flink-docs-setup-cluster/)
* [flink jobmanager high availability](https://ci.apache.org/projects/flink/flink-docs-release-1.6/ops/jobmanager_high_availability.html)

