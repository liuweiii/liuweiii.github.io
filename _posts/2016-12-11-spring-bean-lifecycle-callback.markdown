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

可以通过实现spring的InitializingBean和DisposableBean接口，来影响container的bean管理。container调用afterPropertiesSet()实现前者，调用destroy()实现后者，以确保bean有合适的初始化和析构行为。

### initialization callbacks

org.springframework.beans.factory.InitializingBean接口的afterPropertiesSet方法在container执行完初始化工作后执行。但不建议直接使用InitializingBean接口，因为这样会与Spring耦合。可以使用@PostConstruct，或者，在使用基于XML的配置时，可以使用init-method来指定一个无参方法作为container初始化某个bean后执行的方法。

下面的*code 1*和*code 2*等价的：

*code 1*

```
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```
public class ExampleBean {
  public void init() {
    // do some initialization work
  }
}
```

*code 2*

```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```
public class AnotherExampleBean implements InitializingBean {
  public void afterPropertiesSet() {
    // do some initialization work
  }
}
```
不建议使用*code 2*，因为它会与Spring耦合。