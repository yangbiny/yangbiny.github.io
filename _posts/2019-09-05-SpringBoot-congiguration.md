---
layout: post
title:  "SpringBoot 配置文件加载顺序"
date:   2019-09-05
desc: "Spring,SpringBoot"
keywords: "Spring,SpringBoot,配置文件"
categories: [Spring]
tags: [Spring]
icon: icon-html
---

# 加载位置与优先级
## 项目内部配置文件
springBoot启动时会扫描以下位置的application.properties或者是application.yml文件作为SpringBoot的默认配置文件

```play
–file:./config/（1）
–file:./（2）
–classpath:/config/（3）
–classpath:/（4）
```
优先级为1,2,3,4，由高到低，高优先级会覆盖低优先级的配置文件，如果不冲突就会共同存在

## 外部配置文件
SpringBoot也可以从以下位置加载配置：  

优先级从高到低；  
高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置。  
- 命令行参数
    - 所有的配置都可以在命令行上进行指定；
    - 多个配置用空格分开； --配置项=值  

java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar 
--server.port=8087 --server.context-path=/abc

- 来自java:comp/env的JNDI属性
- Java系统属性（System.getProperties()）
- 操作系统环境变量
- RandomValuePropertySource配置的random.*属性值
- jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件
- jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件
- jar包外部的application.properties或application.yml(不带spring.profile)配置文件
- jar包内部的application.properties或application.yml(不带spring.profile)配置文件
    - 由jar包外向jar包内进行寻找，优先加载带profile的，再加载不带profile的。
- @Configuration注解类上的@PropertySource
- 通过SpringApplication.setDefaultProperties指定的默认属性
