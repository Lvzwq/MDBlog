---
layout: post
title: "SpringBoot配置读取与加载的几种方式"
date: 2017-07-01 23:14:32 +0800
comments: true
categories: Dev
tags: [Java, SpringBoot]
---

> SpringBoot 简化了配置，将原来大量使用xml的配置使用Spring config的方式来是实现，这种方式看上去比xml更加简洁，更好理解。

<!-- more -->

### SpringBoot 配置读取
SpringBoot应用默认读取的应用配置是在application.properties或者application.yml文件中

```
mq.consumer.topic=binlog.unionbusiness.UnionCommodity
mq.consumer.groupId=ad.cps.commission
mq.consumer.address=mq.keeper.service.xxx.org
mq.consumer.batchSize=20
mq.consumer.timeOut=2000

# mq Consumer 
mq.asyn.topic=binlog.unionbusiness.UnionAsynEffectOps
mq.asyn.groupId=unionads.cps.asyneffect
mq.asyn.address=mq.keeper.service.xxx.org
mq.asyn.batchSize=10
mq.asyn.timeOut=2000

# mq Producer
mq.producer.topic=ad.cpscommission.tradeitemid
mq.producer.groupId=ad.cps.commission
mq.producer.address=mq.keeper.service.xxx.org
```
读取的方式有很多种，第一种通过直接通过`@ConfigurationProperties`注解来读取配置

```java
@Slf4j
@Data
@ConfigurationProperties(prefix = "mq.producer")
@Component
public class CorgiProducer implements InitializingBean, DisposableBean{

    private String topic;
    private String groupId;
    private String address;

    private static Producer producer;

...
}
```

另外一种，通过SpringMVC中`@Value("${xxx}")`注解的方式来获取属性文件中的值
```java
@Data
@Component
public class Config {

    @Value("${mq.consumer.groupId}")
    private String groupId;

    @Value("${mq.consumer.topic}")
    private String topic;

    @Value("${mq.consumer.address}")
    private String address;

    @Value("${mq.consumer.batchSize}")
    private int batchSize;

    @Value("${mq.consumer.timeOut}")
    private long timeOut;

...
}
```

通过@Bean结合@ConfigurationProperties的方式可以配置相同类的2个不同实例，如配置2个数据源。
```java
@Bean(name = "defaultConfig")
@ConfigurationProperties(prefix = "mq.consumer")
public CorgiConfig defaultCorgiConfig() {
    return new CorgiConfig();
}


@Bean(name = "asynConfig")
@ConfigurationProperties(prefix = "mq.asyn")
public CorgiConfig asynCorgiConfig() {
    return new CorgiConfig();
}
```





