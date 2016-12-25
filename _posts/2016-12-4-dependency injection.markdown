---
layout: post
title:  "dependency injection"
date:   2016-12-4 19:56:20 +0800
categories: spring
tags:
- learn
- di
- spring
---

*代码示例参考 <a href='https://github.com/liuweiii/spring-demo-di-xml' target='_blank'>spring-demo-di-xml</a>*

#### 1.dependency injection

##### 1.1.Constructor-based dependency injection

**1.1.1 构造方法依赖注入**

1. 在java代码中定义类Bean1和类Bean2，类Bean2是类Bean1的构造函数的参数`Bean1(Bean2 b2)`

2. 在xml中定义构造依赖注入

```
<bean id="bean1" class="x.y.Bean1">
  <constructor-arg name="b2" ref="bean2"/>
</bean>
<bean id="bean2" class="x.y.Bean2"/>
```

**1.1.2 使用工厂方法**

1. 在java代码中定义工厂类Factory和Bean，Factory中的方法创建Bean的实例

```
public class Factory{
  public static Bean createInstance(Param1 p1){
    return new Bean();
  }
}
```

2. 在xml中定义依赖注入

```
<bean id="factory" class="x.y.Factory" factory-method="createInstance">
  <constructor-arg name="p1" .../>
</bean>
```

#### 1.2.Setter-based dependency injection

1. 在java中定义类Bean1和类Bean2，类Bean2是类Bean1点一个属性，且Bean1有个setter方法设置Bean2的值

```
public class Bean1{
  private Bean2 bean2;
  public void setMyBean2(Bean2 b2){
    this.bean2 = b2;
  }
}
```

2.在xml中定义依赖注入

```
<bean id="bean1" class="x.y.Bean1">
  <property name="myBean2" ref="bean2"/>
</bean>
```

#### 1.3.method injection

1. 在java中定义类

```
public abstract class MethodInjection {
  public abstract IBean createBean(); //这种注入方式相当于在这儿的子类
                                      //实现中从 IoC container中获取
                                      //了一个子类的实例

  public void out(int id){
    IBean bean = createBean();
    // process bean
  }
}
...
public class Bean implements IBean {
}
```

2.在xml中定义依赖注入

```
<bean id="methodInjection" class="x.y.MethodInjection">
  <lookup-method bean="bean" name="createBean"/>
</bean>
```
#### 2.注入方式

3种注入方式：

- xml注入
- annotation注入
- java注入

##### 2.1.使用xml注入

略。

##### 2.2.使用annotation注入

##### 2.2.1.配置方式

如下所示：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
				http://www.springframework.org/schema/beans/spring-beans.xsd
				http://www.springframework.org/schema/context
				http://www.springframework.org/schema/context/spring-context.xsd">
	<context:annotation-config/>
</beans>
```

在需要使用annotation注入的bean的xml中，在`<beans>`中加入`<context:annotation-config/>`元素和`xmlns:context`及相应的`xsi:schemaLocation`

##### 2.2.2.常见注解

###### 2.2.2.1. Required

用于bean的setter上，被标注的setter中使用的参数必须在xml配置中在配置阶段配置上，否则会抛出BeanInitializationException异常。

