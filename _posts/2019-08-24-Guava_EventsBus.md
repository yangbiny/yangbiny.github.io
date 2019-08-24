---
layout: post
title:  "EventBus"
date:   2019-08-24
desc: "Guava之EventBus"
keywords: "观察者模式,异步消息"
categories: [Blog]
tags: [Google,Guava,EventBus,观察者模式]
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