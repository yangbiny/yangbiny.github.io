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
springBoot启动时会扫描以下位置的application.properties或者是application.yml文件作为SpringBoot的默认配置文件
```play
–file:./config/
–file:./
–classpath:/config/
–classpath:/
```
