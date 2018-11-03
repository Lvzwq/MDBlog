---
layout: post
title: "Linux上Hadoop安装配置"
date: 2015-11-11 22:56:40 +0800
comments: true
categories: Dev
tags: [Hadoop]
---

<!--more-->
准备工作
安装`java`, 设置java环境

## 下载`Hadoop`
```sh
wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.6.2/hadoop-2.6.2.tar.gz
tar -zxvf  hadoop-2.6.2.tar.gz
cd hadoop-2.6.2
```
## 修改`etc/hadoop/hadoop-env.sh`
将`export JAVA_HOME=${JAVA_HOME}`修改为系统中对应的`Java`环境`export JAVA_HOME=/root/jdk1.7.0_79`

## 伪分布式模式安装
Hadoop可以在单节点上以伪分布式模式运行，是以不同的Java进程模拟分布式运行中的各类节点(NameNode、DataNode、)。
### 配置`Hadoop`
修改以下文件
`core-site.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!-- Put site-specific property overrides in this file. -->
<configuration>
<property>
    <name>fs.default.name</name>
    <value>hdfs://iZ28jq7frf7Z:9000</value> <!--value中为hostname-->
</property>

<property>
   <name>hadoop.tmp.dir</name>
   <value>/root/hadoop/hadooptmpdir</value>
</property>

<property>
  <name>dfs.name.dir</name>
  <value>/root/hadoop/dfsnamedir</value>
</property>
</configuration>
```
`hdfs-site.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!-- Put site-specific property overrides in this file. -->
<configuration>
<property>
    <name>dfs.replication</name>
    <value>1</value>
</property>
</configuration>
```
生成`mapred-site.xml`配置文件。`cp mapred-site.xml.template mapred-site.xml`
```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!-- Put site-specific property overrides in this file. -->
<configuration>
<property>
      <name>mapred.job.tracker</name>
      <value>iZ28jq7frf7Z:9001</value> 
</property>
</configuration>
```
### 设置免密码`SSH`
```sh
ssh-keygen -t rsa
cp id_rsa.pub authorized_keys
```

### 启动`Hadoop`
格式化分布式文件系统
```sh
bin/hadoop namenode -format
```
启动`Hadoop`守护进程
```bash
sbin/start-all.sh
或者
start-dfs.sh 
start-yarn.sh
```
