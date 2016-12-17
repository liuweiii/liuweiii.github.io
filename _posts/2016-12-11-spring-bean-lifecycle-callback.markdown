---
layout: post
title:  "spring bean lifecycle callback"
date:   2016-12-11 19:56:20 +0800
categories: spring
tags:
- learn
- spring
- bean
---

*代码示例参考 <a href='https://github.com/liuweiii/spring-demo-lifecycle-callback' target='_blank'>spring-demo-lifecycle-callback</a>*

bean可以通过实现InitializingBean接口的afterPropertiesSet()方法来执行bean初始化后的操作；
bean可以通过实现DisposableBean接口的destroy()方法来执行bean析构后的操作。

### initialization callbacks

org.springframework.beans.factory.InitializingBean接口的afterPropertiesSet方法在container执行完初始化工作后执行。但不建议直接使用该接口，因为会与Spring耦合。可用@PostConstruct注解，或在XML中用init-method指定一个无参方法。

下面的*code 1*和*code 2*等价的：

*code 1*

```
<bean id="bean" class="a.Bean" init-method="init"/>
```

```
public class Bean {
  public void init() {
    // do some initialization work
  }
}
```

*code 2*

```
<bean id="bean" class="a.Bean"/>
```

```
public class Bean implements InitializingBean {
  public void afterPropertiesSet() {
    // do some initialization work
  }
}
```
不建议使用*code 2*，因为它会与Spring耦合。

### destruction callbacks

org.springframework.beans.factory.DisposableBean接口的destroy方法在包含该bean的container析构后执行。但不建议直接使用该接口，因为会与Spring耦合。可用@PreDestroy注解，或在XML中用destroy-method指定一个无参方法。

下面的*code 3*和*code 4*是等价的：

*code 3*

```
<bean id="bean" class="a.Bean" destroy-method="cleanup"/>
```

```
public class Bean {
  public void cleanup() {
    // do some destruction work (like releasing pooled connections)
  }
}
```

*code 4*

```
<bean id="bean" class="a.Bean"/>
```

```
public class Bean implements DisposableBean {
  public void destroy() {
    // do some destruction work (like releasing pooled connections)
  }
}
```