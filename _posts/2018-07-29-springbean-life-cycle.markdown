---
layout: post
title:  "Spring Bean 生命周期"
date:   2018-07-29 14:01:20 +0800
categories: java
tags:
- spring
---

## 1. 创建

### 1.1 bean的建立

容器查找bean的定义并将其实例化。

### 1.2 bean的属性注入

spring依赖注入，使用bean定义信息配置bean的属性。

### 1.3 设置bean的name

如果bean实现了BeanNameAware接口，工厂调用bean的setBeanName方法。

### 1.4 bean放入工厂

如果bean实现了BeanFactoryAware接口，将调用setBeanFactory方法。

### 1.5 before init

如果bean实现了BeanPostProcessors接口，将调用postProcessBeforeInitialization方法。

### 1.6 init

如果bean实现了InitializingBean接口，将调用afterPropertiesSet方法。

### 1.7 init-method

如果bean定义中指定了init-method，将调用之。

### 1.8 after init

如果bean实现了BeanPostProcessors接口，将调用ProcessaAfterInitialization方法。

## 2. 销毁

### 2.1 destroy

如果实现了DisposableBean接口，将调用destroy方法。

### 2.2 destroy-method

如果bean定义中指定了destroy-method，将调用之。