---
layout: post
title:  "spring tips"
date:   2016-12-3 06:21:20 +0800
categories: java
tags:
- learn
- java
- spring
---

### 1. value/ref/idref

**value**：只能放字符串，可以被转换成int等基本类型。

**ref**：引用定义的其他bean。

**idref**：只能放字符串，该字符串为其他bean的id的字符串值。

### 2. depends-on 标签
```
<bean id="beanDependence" class="com.example.beans.depends.BeanDependence"/>
<bean id="beanHole" class="com.example.beans.depends.BeanHole" depends-on="beanDependence"/>
```
beanDependence会在beanHole前创建，且在beanHole后销毁。及depend-on标签可以控制创建、销毁顺序。