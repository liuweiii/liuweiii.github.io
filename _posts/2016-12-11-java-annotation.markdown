---
layout: post
title:  "java annotation"
date:   2016-12-11 19:56:20 +0800
categories: java
tags:
- learn
- java
- annotation
---

### 1. 作用范围（Target）

**TYPE** 可以放在Class、interface、enum声明上

**FIELD** 可以放在字段或者enum常量声明上

**METHOD** 可以放在方法声明上

**PARAMETER** 可以放在参数声明上

**CONSTRUCTOR** 可以放在构造函数声明上

**LOCAL_VARIABLE** 放在本地变量声明上

**ANNOTATION_TYPE** 放在注解类型声明上，即注解注解的注解，如@Target、@Retention、@Inherited、@Documented

**PACKAGE** 放在包声明上

**TYPE_PARAMETER** since 1.8，Type parameter declaration

**TYPE_USE** since 1.8，use of a type

### 2. 生命周期（Retention）

**源码（SOURCE）** 注解只在源码中存在，编译成.class文件后不存在。Annotations are to be discarded by the compiler。

**class文件中（CLASS）** 在源码和.class文件中都存在，在编译过程中，被保留到class文件中，默认的行为。Annotations are to be recorded in the class file by the compiler but need not be retained by the VM at run time.  This is the default behavior.

**运行时（RUNTIME）** 运行期间起作用，影响运行时逻辑，可以通过反射获取。Annotations are to be recorded in the class file by the compiler and retained by the VM at run time, so they may be read reflectively.

### 3. 使用方法

通过**反射**，找出被某个annotation附加上的类（或其方法、字段等），调用annnotation里的方法获取值进行操作。


![annotation-1](/public/img/2016-12-11-java-annotation-1.jpg)

