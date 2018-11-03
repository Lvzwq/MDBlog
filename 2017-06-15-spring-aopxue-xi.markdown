---
layout: post
title: "Spring AOP学习"
date: 2017-06-15 19:49:18 +0800
comments: true
categories: Dev
tags: [Java]
---

### AOP概念
AOP概念和术语不是Spring独有的，AOP术语并不是直观感受的，而是一种抽象概念。
<!-- more -->
>
切面Aspect: 跨越多个类的模块化的关系。事务管理就是其中一个典型的应用。  
连接点Join Point： 程序运行中的一个点，比如调用方法和处理异常，在Spring AOP中，通常指代方法调用。  
通知Advice: 在一个特殊的Joint Point被Aspect执行的动作。不同的类型的Advice有"around," "before" 和 "after" 类型。  
切入点Pointcut: 通知定义了切面要发生的“故事”和时间，那么切入点就定义了“故事”发生的地点，例如某个类或方法的名称，spring中允许我们方便的用正则表达式来指定  

Spring AOP是用纯Java实现的，不需要特殊的编译过程，并且不需要控制Classloader的层级关系，并且可以用在Servlet容器中和应用服务器中。
Spring AOP目前只支持方法级别的执行Joint Point(在Spring Bean中方法执行的通知Advice)。

#### AOP代理
Spring AOP代理默认使用标准SDK的动态代理，这允许代理任意接口interface。Spring AOP也可以使用CGLIB代理，这是代理类而不是代理接口interface,如果一个类没有实现一个接口默认是使用CGLIB代理的。

#### @AspectJ

需要手动开启@AspectJ支持，可以在Java配置中添加@Configuration上添加@EnableAspectJAutoProxy注解。
```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```
或者在xml配置中开启@AspectJ
```xml
<aop:aspectj-autoproxy/>
```

* 声明一个Aspect
开启了@AspectJ支持，任何Spring bean使用了@Aspect注解都将被Spring检测并配置成Spring AOP。

* 声明一个切入点Pointcut
使用@Pointcut注解的切入点表达式，并且注解的方法必须是含void返回值的方法。
```java
@Pointcut("execution(* transfer(..))")  // the pointcut expression
private void anyOldTransfer() {} // the pointcut signature
```
