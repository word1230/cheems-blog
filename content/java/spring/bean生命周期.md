---
date: '2026-02-21T14:41:01+08:00'
draft: false
slug: '5970346a'
type: posts
title: 'Bean生命周期'
collections: "spring"
categories: [java]
---

bean生命周期，三级缓存解决循环依赖

<!--more-->

## 生命周期

总体上讲分个阶段： 实例化 -> Bean属性填充 -> Bean初始化 -> 就绪 -> 销毁



### 实例化（分为两个阶段：容器启动阶段 和 Bean实例化阶段）

> 容器启动阶段



我们用各种方式来定义bean，这些定义就是元信息

先处理元信息，元信息包括：xml里配置的元信息，注解里配置的元信息，properties文件里的元信息

这些元信息是不同格式。 所以需要需要统一通过BeanDefinationReader将其读入内存



BeanDefinationReader 包含多个实现，

如 xml 使用XmlBeanDefinationReader，

注解 ，使用AnnotatedBeanDefinitionReader



通过这些 Reader， 将元信息读入内存，统一转换为BeanDefination。

这样我们需要创建一个对象时，只需要找到对应的BeanDefination，然后创建对象就可以了



那怎么找呢？ 就需要使用到 BeanDefinationRegister了。 BeanDefinationRegister是一个存储键值对的结构，它存储的是通过特定的`Bean`定义的`id`，映射到相应的`BeanDefination`

当BeandefinationReader 将元信息读入成BeanDefination后，会将其注册到BeanDefinationRegister中，

这样我们通过BeanDefinationRegister 就能找到BeanDefination

 

我们在配置元信息时，会用到占位符，保证一次修改，到处生效。那占位符最终是要替换为真实的数据的，那就需要BeanFactoryPostProcess，最后会对注册到BeanDefinationRegister 中的BeanDefination进行最后的修改



这样容器初始化的过程就结束了。 总结一下： BeanDefinationReader 将 元信息读成BeanDefination，然后将其注册到BeanDefinationRegister， 最后由BeanFactoryPostProcess进行最后修改

> bean 实例化阶段

借助我们前面的BeanDefinationRegister 和 BeanDefination， 使用反射来完成对象的创建



### Bean属性填充

对于基本类型； 如果元信息有配置，就按照元信息里的赋值。 如果没有，在jvm对象实例化时，也会被赋上默认的零值

对于引用对象类型： spring会讲所有已经创建好的对象放入一个map中，spring会检测是否所依赖的对象在map中，如果在，直接注入，如果不在，就先去创建依赖的对象，并将其放入到map中，然后再注入（这里会触发如果依赖问题）



### Bean初始化阶段

这个阶段 会进行四件事情：

1. 调用实现Aware接口的方法

   如：	如果实现了 BeanNameAware,BeanClassLoaderAware， 那就会依次调用setBeanName(), setBeanClassLoader()

2. BeanPostProcessor前置方法：有类实现了BeanPostProcessor，就会调用postProcessBeforeInitialization方法  

3. 初始化

   按序执行： @PostConstruct注解方法，InitializingBean的afterPropertiesSet方法，@Bean指定的initMethod

4. BeanPostProcess后置方法：有类实现了BeanPostProcessor，就会调用postProcessAfterInitialization方法  



### 就绪

然后bean就可以使用了



### 销毁

容器关闭时按顺序执行 @PreDestroy 注解方法、DisposableBean 的 destroy 方法、@Bean 指定的 destroyMethod



## 三级缓存解决循环依赖

### 什么是循环依赖

简单来讲就是 A类依赖了B， B也依赖了A，形成了一个环

比如：

```java
@Component
public class A {
    @Autowired
    private B b; // A 依赖 B
}

@Component
public class B {
    @Autowired
    private A a; // B 依赖 A
}
```

这样就会导致一个问题： 创建A类时，需要进行依赖注入，就需要B，发现B还没有被创建，就先创建B，发现B依赖A，A又没有被创建好，所以又创建A，形成了一个循环，导致无法实例化。

### 三级缓存是如何解决这个问题的

spring 定义了三个map，也就是三级缓存：

1. 第一层：放完全初始化好的Bean
2. 第二层：存放通过反射创建但尚未进行属性注入和初始化的 Bean 实例。用于快速返回早期引用，避免重复从工厂生成。
3. 第三层：存放能生成早期 Bean 引用的工厂。最核心层：当检测到循环依赖时，从此处通过 getObject() 获取早期引用（也就是能存放在第二层的引用）（会触发 getEarlyBeanReference() 处理 AOP 代理等逻辑）。



三级缓存中的工厂的所用： 如果需要 AOP，它负责创建 A 的代理对象；如果不需要，它返回 A 的原始对象。

获取 Bean 时，按顺序检查： 一级缓存 → 二级缓存 → 三级缓存。

#### 解决过程

创建BeanA

spring实例化A ，将A的工厂对象放入到三级缓存，但此时还没有属性注入

进行属性注入时，发现需要B，去缓存找，没有找到，于是实例化B



实例化B

spring实例化B， 将B的工厂对象放入三级缓存

进行属性注入时，发现需要A

去一级缓存找，没有

二级缓存找，还没有

三级缓存找到A的工厂对象

执行工厂方法：getObject。  这里如果A需要AOP，则生成代理对象，不需要，则原始对象

将得到的对象放入二级缓存

从三级缓存移除A的工厂（避免重复创建）

将A的引用给B

B的属性注入完成，初始化成功，将B放入一级缓存中



回到A，完成B的属性注入，完成了A的初始化，将A放入一级缓存。

### 为什么是三级

答案：为了保证AOP代理的一致性



> 如果只有两级缓存

当A实例化时，将A的原始对象放入到 二级缓存中。

A进行依赖注入，发现需要B



B进行实例化，将B的原始对象放入二级缓存

B进行属性注入，发现需要A

找到二级缓存中的==原始对象== 进行属性注入

B初始化完成，将B放入一级缓存



回到A，完成了属性注入。

在初始化时，出发了AOP逻辑，spring创建了A的代理对象

将A的代理对象放入了一级缓存



此时 ==b的属性注入时A的原始对象，而spring容器最终放的是A的代理对象，两个不一致了==



> 如果是三级缓存

当A实例化时，将A的工厂对象放入到 三级缓存中。

A进行依赖注入，发现需要B



B进行实例化，将B的工厂对象放入三级缓存

B进行属性注入，发现需要A

==找到三级缓存中的A的工厂对象，发现需要AOP代理，于是生成了代理对象==，

将A的代理对象放入到B中



这样就解决了问题。







### 三级缓存不能解决循环依赖的情况

1. **单例 (Singleton) + 属性注入 (Setter/Field)：** Spring **可以** 解决。

2. 单例 (Singleton) + 构造器注入 (Constructor)：Spring 无法 解决。

- 原因：创建 A 时需要先创建 B，创建 B 时需要先创建 A，双方都在等待对方实例化完成，导致死锁，抛出 `BeanCurrentlyInCreationException`。

3. 原型 (Prototype)： Spring 无法 解决。

- 原因：原型 Bean 每次请求都会创建新实例，缓存机制不适用。



为什么构造器不行：

**三级缓存的介入时机：** Spring 是在 **（实例化）完成之后**，**（属性填充）开始之前**，将 Bean 的工厂对象放入三级缓存的。

构造器的循环依赖都完不成实例化这一步



1. Spring 尝试实例化 A
   - 进入 `createBean` 流程。
   - 准备调用 A 的构造函数 `new A(b)`。
   - **卡住了！** 构造函数需要参数 `b`。Spring 必须先去创建 B。
2. Spring 尝试实例化 B
   - 进入 `createBean` 流程。
   - 准备调用 B 的构造函数 `new B(a)`。
   - **卡住了！** 构造函数需要参数 `a`。Spring 必须先去创建 A。
3. 死锁 (Deadlock)
   - A 等 B 创建好才能实例化。
   - B 等 A 创建好才能实例化。
   - **结果：** 两个 Bean 连 **实例化（new 对象）** 这一步都没完成。



#### 必须要使用构造器注入，如何解决这个问题

1. 使用@lazy注解

```java
@Component
public class A {
    private final B b;
    // Spring 会注入一个 B 的代理对象，实际调用时才会去获取真正的 B
    public A(@Lazy B b) { 
        this.b = b;
    }
}
```

*原理：* `@Lazy` 会让 Spring 注入一个代理对象，而不是直接创建 B。这样 A 的构造函数可以立即完成实例化，A 就能放入缓存，B 创建时就能从缓存拿到 A，从而打破死锁。



