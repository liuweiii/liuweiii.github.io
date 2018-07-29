---
layout: post
title:  "distributed system"
date:   2018-07-29 14:01:20 +0800
categories: design
tags:
- disturibute
---
### 1 什么是分布式系统

多个计算单元通过网络连接起来的系统。

### 2 分布式系统需要考虑的特性

#### 2.1 可扩展性scalability

系统可以在计算资源之间均衡负载，并且一部分资源工作，其余部分待命（即可以在系统负载过高时，将待命的部分用于分担负载）。

#### 2.2 可用性availablity

系统可以在节点或网络产生一些异常时，仍然提供服务。

#### 2.3 一致性consistency

如系统中存在多个数据的备份，它们需要保持一致。

#### 2.4 安全性security

系统提供多用户权限控制。

### 3. CAP

一致性、可用性、分区性的折中。

### 4. ACID/BASE

事务的强一致性和弱一致性，一般分布式是BASE，本地可以达到ACID。

### 5. TCC/XA/2PC/3PC/Paxos

事务一致性算法。

