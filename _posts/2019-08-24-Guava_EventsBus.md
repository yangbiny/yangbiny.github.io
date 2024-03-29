---
title:  "EventBus"
date:   2019-08-24
desc: "Guava之EventBus"
keywords: "观察者模式,异步消息"
categories: [Blog]
tags: [google,guava,event-bus]
icon: icon-html
---

# 概述
EventBus是由Google-Guava实现的一种消息发布-订阅的类库，实现了观察者模式，通过EventBus通知注册/注销订阅者，
发送信息给观察者。

# 组成部分
消息负责人：用于注册/注销/通知订阅者/观察者。  
订阅者：注册到EventBus中，通过使用注解@Subscribe表明这个方法接受监听。

# 案例

消息负责人：
```java
import com.google.common.eventbus.EventBus;

public class EventBusCenter {

  private static EventBus eventBus = new EventBus();

  private EventBusCenter() {

  }

  public static EventBus getInstance() {
    return eventBus;
  }

  // 注册
  public static void register(Object object) {
    eventBus.register(object);
  }

  //注销
  public static void unregister(Object object) {
    eventBus.unregister(object);
  }

  // 通知
  public static void post(Object object) {
    eventBus.post(object);
  }

}
```
观察者
```java
import com.google.common.eventbus.Subscribe;
/**
 * 观察者1
 */
public class DataObserver1 {

  // 表明这个方法接受监听，如果不添加，不会接受到信息
  @Subscribe
  public void func(String msg) {
    System.out.println("DateObserver1 : " + msg);
  }
}
```
```java
import com.google.common.eventbus.Subscribe;

/**
 * 观察者2
 */
public class DataObserver2 {

  @Subscribe
  public void func(Integer msg) {
    System.out.println("DateObserver2 : " + msg);
  }

  @Subscribe
  public void func(String msg) {
    System.out.println("DataOnserver2 : " + msg);
  }
}

```

观察者会根据EventBus.post()传入的参数类型，通知相关类型的观察者。

```java
import cn.pzhu.impassive.logis.eventbus.DataObserver1;
import cn.pzhu.impassive.logis.eventbus.DataObserver2;
import cn.pzhu.impassive.logis.eventbus.EventBusCenter;

public class Application {

  public static void main(String[] args) {
    DataObserver1 observer1 = new DataObserver1();
    DataObserver2 observer2 = new DataObserver2();

    EventBusCenter.register(observer1);
    EventBusCenter.register(observer2);

    System.out.println("=========================================");

    EventBusCenter.post("test");
    EventBusCenter.post(123);
  }
  
}
```

# 实现原理

当调用register的时候，会解析这一个对象，通过反射解析出改对象中的方法的信息，会将所有属
于相同类型的监听者放在一个监听器中。使用的是Multimap，与传统的Map相比，实现了一对多的
存储。调用post的时候，逐个遍历并执行（根据参数类型）
