---
layout: post
title:  "深入java虚拟机-3 类型的生命周期"
date:   2016-07-11 22:03:20 +0800
categories: java
tags:
- learn
- java
---

## 1 类型装载、连接与初始化

![inside-jvm-3](/public/img/2016-07-11-inside-jvm3.png)

### 1.1 装载

装载的最终产品就是一个java.lang.Class类的实例对象。

1. 加载类型二进制数据
   - 从本地文件系统装载一个Java class文件
   - 通过网络下载一个Java class文件
   - 从一个ZIP、JAR、CAB或者其他某种归档文件中提取Java class文件
   - 从一个i额专有数据库中提取Java class文件
   - 把一个i额Java源文件动态编译为class文件格式
   - 动态为某个类型计算其class文件数据
   - 使用上述任何方法，但是使用不同于Java class文件的其他二进制文件格式
2. 创建类型
   把一个类型的二进制数据解析为方法区中的内部数据结构
3. 在堆上建立一个Class对象

如果一个类装载器在预先装载时遇到缺失或者错误的class文件，它必须等到程序首次主动使用该类时才报告错误（LinkageError异常）。

### 1.2 验证

确认类型符合Java语言的语义，并且它不会危及虚拟机的完整性。