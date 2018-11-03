---
layout: post
title: "ElasticSearch5.3.1 集群搭建"
date: 2017-05-06 16:25:43 +0800
comments: true
categories: Dev
tags: [ElasticSearch]
---


参考: [Elasticsearch5.2.1集群搭建，动态加入节点，并添加监控诊断插件](http://blog.csdn.net/gamer_gyt/article/details/59077189)

<!--more-->

## 准备阶段
1、Java1.8  
2、下载elasticsearch5.3.1    
3、设置环境变量 JAVA_HOME


## 配置修改

修改`elasticsearch.yml`

```
cluster.name: es5-cluster   # 集群名称
node.name: node-1    # 节点名称
network.host: 192.168.138.129
http.port: 9200
transport.tcp.port: 9300
node.master: true  # 是否作为集群的主结点
node.data: true  # 是否存储数据，值为true
discovery.zen.ping.unicast.hosts: ["192.168.138.128:9300", "192.168.138.129:9300", "192.168.138.130:9300"]
discovery.zen.minimum_master_nodes: 1
gateway.recover_after_nodes: 2
gateway.recover_after_time: 5m
gateway.expected_nodes: 1
script.engine.groovy.inline.search: on
script.engine.groovy.inline.aggs: on
indices.recovery.max_bytes_per_sec: 20mb
```

## 常见的错误
启动命令`bin/elasticsearch -d`后，出现

```
[2017-04-22T10:14:51,251][ERROR][o.e.b.Bootstrap          ] [node-3] node validation exception
bootstrap checks failed
max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
[2017-04-22T10:14:51,332][INFO ][o.e.n.Node               ] [node-3] stopping ...
[2017-04-22T10:14:51,401][INFO ][o.e.n.Node               ] [node-3] stopped
[2017-04-22T10:14:51,402][INFO ][o.e.n.Node               ] [node-3] closing ...
[2017-04-22T10:14:51,417][INFO ][o.e.n.Node               ] [node-3] closed
```

解决方法

```
sudo sysctl -w vm.max_map_count=655360
# 设置系统打开文件最大句柄
sudo cat /proc/sys/fs/file-max
ulimit -n 65536
```

可以根据服务器的配置修改JVM参数，修改文件`elasticsearch-5.3.1/config/jvm.options`

```
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms512m
-Xmx512m

################################################################
## Expert settings
################################################################
##
## All settings below this section are considered
## expert settings. Don't tamper with them unless
## you understand what you are doing
##
################################################################

## GC configuration
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFraction=75
-XX:+UseCMSInitiatingOccupancyOnly
```

打开浏览器，访问http://192.168.138.129:9200, 如果出现以下页面则表示安装成功
![http://zhangwenqiang.com.cn/WX20170506-164120@2x.png](http://zhangwenqiang.com.cn/WX20170506-164120@2x.png)

访问http://192.168.138.129:9200/_cluster/health?pretty=true
![http://zhangwenqiang.com.cn/WX20170506-164430@2x.png](http://zhangwenqiang.com.cn/WX20170506-164430@2x.png)

## kibana安装
```
wget https://artifacts.elastic.co/downloads/kibana/kibana-5.3.1-linux-x86_64.tar.gz
```

修改config/kibana.yml文件

```
server.port: 5601
server.host: "192.168.138.129"
elasticsearch.url: "http://192.168.138.129:9200"
```

启动

```
bin/kibana 
```

访问http://192.168.138.129:5601 客户端


##head插件安装
下载head插件，首先需要保证系统安装nodejs

```
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install
```

在elasticsearch.yml文件中添加

```
# 增加新的参数，这样head插件可以访问es
http.cors.enabled: true
http.cors.allow-origin: "*"
```

修改head/Gruntfile.js

```
connect: {
    server: {
        options: {
            port: 9100,
            hostname: '*',
            base: '.',
            keepalive: true
        }
    }
}
```

把localhost修改成你es的服务器地址，如:

```
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://192.168.138.129:9200";
```

启动grunt服务

```
node_modules/grunt/bin/grunt server
```

访问http://192.168.138.129:9100, 如果出现以下则表示集群安装成功

![http://zhangwenqiang.com.cn/WX20170506-171946@2x.png](http://zhangwenqiang.com.cn/WX20170506-171946@2x.png)

