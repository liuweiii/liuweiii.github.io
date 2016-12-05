---
layout: post
title:  "bean scope"
date:   2016-12-4 19:56:20 +0800
categories: spring
tags:
- learn
- scope
- spring
---

*代码示例参考 <a href='https://github.com/liuweiii/spring-demo-bean-scope' target='_blank'>spring-demo-bean-scope</a>*

### 1. singleton scope(default)

每次从同一个IoC container中获取同一个id的bean，都是同一个实例

```
<bean id="bean" scope="singleton" p:name="injectedName"/>
```
在java中

```
Bean bean = context.getBean("bean", Bean.class);
System.out.println(bean.getName()); //the name is injectedName
bean.setName("name1"); 
Bean bean1 = context.getBean("bean", Bean.class);
System.out.println(bean1.getName()); //the name is name1!!!
```

### 2.prototype scope

每次从同一个IoC container中获取同一个id的bean，都是不同实例

```
<bean id="bean" scope="prototype" p:name="injectedName"/>
```
在java中

```
Bean bean = context.getBean("bean", Bean.class);
System.out.println(bean.getName());//the name is injectedName
bean.setName("prototype1");
Bean bean1 = context.getBean("bean", Bean.class);
System.out.println(bean.getName());//the name is injectedName!!!
```