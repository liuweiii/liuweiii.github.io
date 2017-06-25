---
layout: post
title:  "java ConcurrentHashMap"
date:   2017-06-25 21:56:20 +0800
categories: java
tags:
- learn
- java
- ConcurrentHashMap
---
### 1. JDK1.7中的实现

![concurrenthashmap-1](/public/img/2017-06-25-concurrenthashmap.png)

#### 1.初始化ConcurrentHashMap
ConcurrentHashMap内部有一个Segment数组。每个Segment相当于一个HashMap，里面有一个HashEntry数组（HashEntry是一个链表）。

ConcurrentHashMap初始化需要3个参数：
 1. initialCapacity：初始化容量
 2. loadFactor：加载因子
 3. concurrencyLevel：并行等级

Segment数组的长度(ssize)为大于等于concurrencyLevel的最小的2的n次方，如concurrencyLevel为5，ssize就为8，concurrencyLevel为16，ssize就为16。

每个Segment里HashEntry数组的长度(cap)都是一样的，该长度(cap)满足:
1. cap至少为2
2. cap为2的n次方
3. cap大于等于initialCapacity/ssize的结果向上取整

如 initialCapacity=5, concurrencyLevel=3,则ssize=4, initialCapacity/ssize向上取整=2,cap=2。

jdk1.7只初始化了第一个Segment及里面的HashEntry数组。

#### 2. put／remove／get

先定为到具体的Segment，然后再从里面put/remove/get

put时如果当前Segment里的元素个数超过一定大小，当前Segment里的HashEntry数组会扩容，当前Segment里的元素重新hash。整个ConcurrentHashMap的Segment数组长度是不变的。put进来的元素会放到当前Segment链表的首位。

remove时，会讲当前Segment里被remove的节点之前的节点都复制一遍，跟被remove掉的节点后面的节点连起来。如此做的原因是Segment里的HashEntry的next、key都是final的，不能被修改。如此做的原因是为了提高并发get效率。

### 2. JDK 1.8中的实现

#### 1.不使用分段（segment）方式，直接用一个HashEntry数组（table）来存储元素

#### 2.table中的每个元素要么是一个链表，要么是一个红黑树（链表长度大于8时转换）

#### 3.synchronized table中某个HashEntry链表（树）的首节点操作该链表（树）里的元素


