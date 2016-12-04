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

#### 1.Constructor-based dependency injection

**1.1 真正的构造方法依赖注入**

1. 在java代码中定义类Bean1和类Bean2，类Bean2是类Bean1的构造函数的参数（`public Bean1(Bean2 b2)`）

2. 在xml中定义构造依赖注入

```
<bean id="bean1" class="x.y.Bean1">
  <constructor-arg name="b2" ref="bean2"/>
</bean>
<bean id="bean2" class="x.y.Bean2"/>
```

**1.2 使用工厂方法**

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

#### 2.Setter-based dependency injection

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

#### 3.method injection

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