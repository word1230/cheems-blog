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

### 候选类
需要知道有哪些候选

springboot2.7之前 是从 `META-INF/spring.factories` 文件的键 `org.springframework.boot.autoconfigure.EnableAutoConfiguration`中读取.

`META-INF/spring.factories` 是键值对的形式

spring2.7之后是从**`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`**
文件中读取

每一行就是一个自动配置类的全限定名，更简洁且易于维护。

### 按需生效

加载全部候选类只是第一步，真正用到的是通过 **条件注解** 过滤出的配置。

`AutoConfigurationImportSelector` 内部会调用 `filter(configurations, autoConfigurationMetadata)`，利用 Spring 的 `@Conditional` 机制判断每个配置类是否满足条件。常用的条件注解有：

|注解|作用|
|---|---|
|`@ConditionalOnClass`|某些类在 classpath 中存在时才生效|
|`@ConditionalOnMissingClass`|某些类不在 classpath 时生效|
|`@ConditionalOnBean`|容器中存在指定 Bean 时才生效|
|`@ConditionalOnMissingBean`|容器中不存在指定 Bean 时生效|
|`@ConditionalOnProperty`|配置文件中存在指定属性且值匹配时生效|
|`@ConditionalOnResource`|指定资源文件存在时生效|
|`@ConditionalOnWebApplication`|当前为 Web 应用时生效|
|`@ConditionalOnExpression`|SpEL 表达式结果为 true 时生效|
过滤后的配置类被 `ImportSelector` 以字符串数组形式返回，Spring 容器将它们当作普通的 `@Configuration` 类进行处理：



### 总结

1. 启动类上的 `@SpringBootApplication` → `@EnableAutoConfiguration`。
    
2. `@Import(AutoConfigurationImportSelector.class)` 执行。
    
3. 从类路径的 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`（或旧的 `spring.factories`）读取所有候选自动配置类名称。
    
4. 结合各配置类上的条件注解（`@ConditionalOnClass`、`@ConditionalOnMissingBean` 等）进行过滤，仅保留当前环境满足条件的配置类。
    
5. 将这些配置类注册为 Spring 的 Bean 定义。
    
6. 容器在后续刷新时解析配置类，创建对应的 Bean，完成自动配置。


## 让自己的类被自动配置


一般来讲就是通过ioc 容器, 将自己的类托管给容器就行, 也就是用@Component, @Bean 等

如果提供自己的类给其他人使用


