---
layout: post
title: " Java 对字符串转为可执行代码"
date: 2017-04-25 00:54:14 +0800
comments: true
categories: Dev
tags: [Java]
---

<!--more-->

## 方法一
参考: [java实现字符串转换成可执行代码](http://wiselyman.iteye.com/blog/1677444)



使用commons的jexl包

```xml
 	<dependency>
          <groupId>org.apache.commons</groupId>
          <artifactId>commons-jexl</artifactId>
          <version>2.0</version>
  </dependency>
```
代码示例

```java
Map<String,Object> map = new HashMap<String,Object>();  
map.put("testService",testService);  
map.put("person", person);  
String expression="testService.save(person)";  

JexlEngine jexl=new JexlEngine();  
Expression e = jexl.createExpression(expression);  
JexlContext jc = new MapContext();  
for(String key:map.keySet()){  
      jc.set(key, map.get(key));  
}  
Object o = e.evaluate(jc);
```

## 方式二
参考:[java执行字符串数学表达式 ScriptEngine](http://blog.csdn.net/w1014074794/article/details/45968559)

使用Java自带的jdk, `javax.script.ScriptEngine`

```java
  String str = "a&&b&&c";
  ScriptEngineManager factory = new ScriptEngineManager();
  ScriptEngine engine = factory.getEngineByName("JavaScript");
  str = str.replace("a", "true");
  str = str.replace("b", "true");
  str = str.replace("c", "false");
  Object o = null;
  try {
      o = engine.eval(str);
  } catch (ScriptException e) {
      e.printStackTrace();
  }finally {
      System.out.println("o = " + o);
  }
```
