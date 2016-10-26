---
layout: post
title:  "java memory model and thread specification"
date:   2016-10-26 06:21:20 +0800
categories: java
tags:
- learn
- java
---

## 1.Introduction

只能通过Thread类创建线程对象。

同一个对象的多个synchronized执行体中如果有一个synchronized被进入，其他的也会被lock（相当于整个object被lock）。

不管synchronized的执行体是正常还是异常结束，都会自动unlock。