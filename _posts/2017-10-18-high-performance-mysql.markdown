---
layout: post
title:  "high performance mysql"
date:   2016-02-28 18:20:20 +0800
categories: mysql
tags:
- learn
- mysql
---

* 1. 搜索的时候如果有多个条件，在这些条件上创建多个单列索引不如创建一个适合顺序的复合索引，正确的索引列顺序依赖于使用该索引的查询，并且同时需要考虑如何更好地满足排序和分组的需求。