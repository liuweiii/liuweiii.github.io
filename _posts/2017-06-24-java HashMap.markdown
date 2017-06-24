---
layout: post
title:  "java HashMap"
date:   2017-06-24 20:56:20 +0800
categories: java
tags:
- learn
- java
- utils
---

### 1.概述
HashMap内部由一个`Node<K,V>[]`的table管理。table的每一个元素即所谓的bucket，同一个bucket的hashcode是一样的。如果用无参构造函数new一个HashMap，table的长度为默认值16。可以在构造函数中指定长度，但是HashMap会将table的长度初始化为大于等于该值的最小的2的n次方。如指定长度为3，则table长度为4，指定长度为17，table长度为32。

往HashMap中添加元素时，如果存放该元素的bucket不为空（为空直接添加进去），会将该元素放到该bucket的链表中。java8中，如果链表长度大于8，会把该链表（查找复杂度为O(n)）转换为红黑树(查找复杂度为log(n))存储。

当HashMap里存放的元素个数大于某个阈值（该阈值为loadFactor*capacity）时，table会resize（即增长），table长度扩大为原来的2倍，扩大后会将原table中的元素全部重新散列到扩大后的table中。在java8中，如果某个bucket里时红黑树且其节点个数少于6，会把该树转换为链表存储。


### 2.重要属性

- capacity：HashMap的容量，table的长度，默认为16
- loadFactory:负载因子，默认为0.75
- threshold：扩容阈值，等于capacity*loadFactory
- size：元素个数
- keySet：返回key的集合
- values：返回value的集合
- entrySet：返回entry的集合，即元素的集合（HashMap内部使用Node表示元素，对外使用entry表示）
- modcount：被修改的次数
 

### 3.put
 1. 元素的key的hashcode混淆（*见下文 6.hash*）后和capacity-1做与运算，得出的值为该key在HashMap中最终的hash值（即table中的位置，即该被放入的bucket中table中的位置）；
 2. 如果该bucket为null，直接把该元素放入该bucket；
 3. 如果该bucket不为null，将该元素放入该bucket链表末尾，若该链表长度超过8，变换该链表为红黑树；
 4. 如果size（即元素个数）大于threshold，执行resize扩容。

### 4.get
1. key的hashcode混淆后和capacity-1做与运算，得出该key所在的bucket
2. 若该bucket的key与该key相等，直接返回该bucket
3. 若该bucket的key与该key不等，在该key的链表（或红黑树）中查找该值

### 5.resize
1. 正常情况（当前capacity小于int类型的最大值的一半）下会重新new一个原table两倍大的table，并将原table中的数据移动（重新散列）到新table中；
2. 若当前要移动的元素的key的hash（混淆后的hash）与新hash（混淆后的）值相等，保持当前位置；
3. 若当前要移动的元素的key的hash（混淆后的hash）与新hash（混淆后的）值不等，移动到`当前位置+原capacity`的位置；
4. 若当前capacity大于int类型最大值的一半，直接返回int类型的最大值为capacity。
5. 若table为null，初始化table。

### 6.hash混淆
jdk害怕有些人实现的hashcode散列的不够均匀，又搞了个方法帮大家再散列一次。在jdk8以前是把key的hashcode的某些位相互异或里几次，jdk8里简化为高16位与低16位异或一次。后和capacity-1做与运算（即取出当前capacity二进制位数相同的位数），得出的值为该key在HashMap中最终的hash值（即table中的位置，即该被放入的bucket中table中的位置）。

### 7.fail-fast
当执行put、clean等操作时，会让modcount+1。执行foreach、next等操作时，会先记录modcount等值，待操作完后再与之前记录的值比较，如果不相等，说明被修改过，就抛出ConcurrentModificationException。