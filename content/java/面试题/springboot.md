---
date: 2026-04-16T09:19:16+08:00
draft: false
slug: 7628bd7d
type: posts
title: Springboot
collections: 面试题
categories:
  - java
---

<!--more-->

## 自动装配的原理

### 加载入口
核心入口:@springbootApplication
这是组合注解 包含多个注解
真正触发自动装配的是@EnableAutoConfiguration注解
关键在于@Import(AutoConfigurationImportSelector.class)

导入的这个类AutoConfigurationImportSelector.class实现了`ImportSelector`接口

负责把生效的自动配置类注册到容器中,.

### 从哪里加载
具体从哪里加载呢:

springboot2.7之前 是从 `META-INF/spring.factories` 文件的键 `org.springframework.boot.autoconfigure.EnableAutoConfiguration`中读取.

`META-INF/spring.factories` 是键值对的形式

spring2.7之后是从**`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`**
文件中读取

每一行就是一个自动配置类的全限定名，更简洁且易于维护。

### 按需生效

加载全部候选类只是第一步，真正用到的是通过 **条件注解** 过滤出的配置。