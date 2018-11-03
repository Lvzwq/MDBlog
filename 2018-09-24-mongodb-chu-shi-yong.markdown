---
layout: post
title: "Java原生API操作MongoDB"
date: 2018-09-24 01:16:53 +0800
comments: true
categories: Dev
tag: [Java,MongoDB]
---

本文主要介绍如何用原生api简易操作mongodb数据库

### 加入依赖

```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.8.2</version>
</dependency>
```

<!--more -->

使用集群方式连接`Mongodb`

### Java操作连接

```java
MongoCredential credential = MongoCredential.createCredential(MONGO_USRE, "admin", MONGO_PASSWD.toCharArray());
MongoClientOptions options = MongoClientOptions.builder()  
    .serverSelectionTimeout(30000)  
    .threadsAllowedToBlockForConnectionMultiplier(50)  
    .maxWaitTime(6000)  
    .sslEnabled(false)  
    .build();
MongoClient mongoClient = new MongoClient(Arrays.asList(
        new ServerAddress("10.50.xx.1", 27017),
        new ServerAddress("10.50.xx.2", 27017),
        new ServerAddress("10.xx.3.", 27017)), credential, options);

// MongoDatabase 是一个不可变类型
MongoDatabase database = mongoClient.getDatabase(DBNAME);
```

### 创建文档连接

```java
//如果根据实体类的形式插入和查询，需要实例化编解码器CodecRegistry

import static org.bson.codecs.configuration.CodecRegistries.fromProviders;  
import static org.bson.codecs.configuration.CodecRegistries.fromRegistries;

MongoDatabase database = MongoClientManager.getInstance().getDataBase("ad");
CodecRegistry pojoCodecRegistry = fromRegistries(MongoClientSettings.getDefaultCodecRegistry(),
              fromProviders(PojoCodecProvider.builder().automatic(true).build()));

// 存储结构实例类ChannelConsumeEntity
MongoCollection<ChannelConsumeEntity> collection = database.getCollection("channelConsume", ChannelConsumeEntity.class).withCodecRegistry(pojoCodecRegistry);

MongoCollection<Document> collection = database.getCollection("channelConsume", Document.class).withCodecRegistry(pojoCodecRegistry);
```

### 插入及更新

```java
ChannelConsumeEntity entity = new ChannelConsumeEntity();  
entity.setBusinessType(value.getBusinessType());  
entity.setChannel(value.getChannel());  
entity.setDate(date);  
entity.setHour(hour);  
entity.setConsume(value.getConsume());  
entity.setVirtualConsume(value.getVirtualConsume());  
entity.setHit(value.getCount());   
collection.insertOne(entity);
// 批量插入
collection.insertMany();
```

#### 更新操作

原生`SQL`操作修改`mongodb`  

```SQL
// 批量更新文档：
db.collection.update(
<query>,
<update>,
{
upsert: <boolean>,
multi: <boolean>,
writeConcern: <document>
}
)
- query : update的查询条件，类似sql update查询内where后面的。
- update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
- upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- writeConcern :可选，抛出异常的级别。
```

```java
// 原生 mongodb SQL语法
// db.channelConsume.update({"date": xx, "hour": xx, }, {"$set": {"consume": xx, "virtualConsume":xx}})

collection.updateOne(new Document("date", date)  
    .append("hour", hour)  
    .append("businessType", value.getBusinessType())  
    .append("categoryId", value.getCategoryId()),  
    new Document("$set", new Document("hit", entity.getHit() + value.getCount())  
    .append("consume", entity.getConsume() + value.getConsume())  
    .append("virtualConsume", entity.getVirtualConsume() + value.getVirtualConsume())));

// 更新操作
collection.updateOne(new Document("date", date)  
    .append("hour", hour)  
    .append("businessType", value.getBusinessType())  
    .append("channel", value.getChannel())  
    .append("detail.minute", minute),  
    new Document("$set", new Document("detail.$.hit", rawData.getHit() + value.getCount())  
    .append("detail.$.consume", rawData.getConsume() + value.getConsume())  
    .append("detail.$.virtualConsume", rawData.getVirtualConsume() + value.getVirtualConsume())  
 ));
```

### 查询

```java
FindIterable<ChannelConsumeEntity> findIterable = collection.find(new Document("date", date)  
  .append("businessType", value.getBusinessType())  
  .append("channel", value.getChannel())  
  .append("hour", hour), ChannelConsumeEntity.class).limit(1);  

ChannelConsumeEntity entity = findIterable.first();
// 或者循环方式遍历
for (ChannelConsumeEntity entity: findIterable) {

}
```

### 聚合操作

管道的概念：管道在Unix和Linux中一般用于将当前命令的输出结果作为下一个命令的参数。
MongoDB的聚合管道将MongoDB文档在一个管道处理完毕后将结果传递给下一个管道处理。管道操作是可以重复的。
表达式：处理输入文档并输出。表达式是无状态的，只能用于计算当前聚合管道的文档，不能处理其它的文档。

这里我们介绍一下聚合框架中常用的几个操作：

- `$project`：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
- `$match`：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
- `$limit`：用来限制MongoDB聚合管道返回的文档数。
- `$skip`：在聚合管道中跳过指定数量的文档，并返回余下的文档。
- `$unwind`：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
- `$group`：将集合中的文档分组，可用于统计结果。
- `$sort`：将输入文档排序后输出。
- `$geoNear`：输出接近某一地理位置的有序文档。

```SQL
# 取出20181001号按照categoryId进行聚合的数据，类似于mysql如下
select SUM(payOrders) AS payOrderSum, SUM(payPrice) AS payPrice, categoryId, date FROM categoryOrder
 WHERE date = 20181001 group by categoryId order by payPrice DESC;

# mongodb sql语法
db.categoryOrder.aggregate({"$match": {"date": 20181001}}, 
{"$group":{"_id":{"categoryId": "$categoryId"}, "payOrderSum":{"$sum":"$payOrders"}, "payPrice":{"$sum": "$payPrice"}}}, 
{"$sort": {"payPrice": -1}})
```

```java
MongoCollection<ChannelConsumeEntity> collection = database.getCollection("channelConsume", ChannelConsumeEntity.class).withCodecRegistry(pojoCodecRegistry);  

List<BasicDBObject> pipeLine = new ArrayList<>(3);  
BasicDBObject match = new BasicDBObject("$match", new Document("date", date));  
BasicDBObject group = new BasicDBObject("$group", new Document("_id", new Document("channel", "$channel").append("businessType", "$businessType"))  
  .append("consume", new Document("$sum", "$consume"))  
  .append("virtualConsume", new Document("$sum", "$virtualConsume"))  
  .append("hit", new Document("$sum", "$hit")));  
BasicDBObject sort = new BasicDBObject("$sort", new Document("hit", -1));  

pipeLine.add(match);  
pipeLine.add(group);  
pipeLine.add(sort);  
AggregateIterable<Document> aggregateIterable = collection.aggregate(pipeLine, Document.class);
```

### 参考

* [MongoDB POJO Support](http://rosslawley.co.uk/mongodb-pojo-support/)
* [菜鸟文档 - MongoDB 更新文档](http://www.runoob.com/mongodb/mongodb-update.html)
* [菜鸟文档 - MongoDB聚合](http://www.runoob.com/mongodb/mongodb-aggregate.html)
