---
date: '2026-02-24T09:53:28+08:00'
draft: false
slug: '50628da4'
type: posts
title: 'AOP进阶'
collections: "spring"
categories: [java]
---

<!--more-->

## 代理

AOP 就是通过代理来实现的

<br/>

代理又分

1. 静态代理
2. 动态代理
3. cglib代理



### 静态代理

特征：在编译时就已经写好了

比如：

<br/>

写一个接口

```java
public interface UserService {
    void save();
}
```

它的实现

```java
public class UserServiceImpl implements UserService {

    @Override
    public void save() {
        System.out.println("保持成功");
    }
    
}
```

写一个代理类

```java
public class UserServiceProxy implements UserService {


    private UserService userService;

    public UserServiceProxy(UserService userService){
        this.userService = userService;
    }

    @Override
    public void save() {
       System.out.println("开始事务");
       userService.save();
       System.out.println("事务结束");
    }

}
```

使用时：

```java
UserService target = new UserServiceImpl();
UserService proxy = new UserServiceProxy(target);
proxy.save();
```



### 动态代理

特征： 运行时生成代理类

java中有两种实现：

- jdk动态代理
- cglib动态代理



这两个的区别：

- jdk 代理

  - 条件： 目标实现了接口
  - 基于Proxy类实现的
  - 只能代理接口中的方法

- cglib动态代理

  - 条件：基于继承

  - 原理：基于 ASM 字节码生成库，生成目标类的**子类**。 重写方法，在方法前后插入增强逻辑

  - 不需要接口

    通过继承实现

    不能代理 final 类

    不能代理 final 方法



#### jdk动态代理

例如：

写一个接口

```java
public interface UserService {
    void save();
}
```

它的实现

```java
public class UserServiceImpl implements UserService {

    @Override
    public void save() {
        System.out.println("保持成功");
    }
    
}
```

创建invocationHandler

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class MyInvocationHandler implements InvocationHandler {

    private Object target;

    public MyInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("开始事务");
        Object invoke = method.invoke(target, args);
        System.out.println("结束事务");
        return invoke;
    }

    public static void main(String[] args) {
        UserServiceImpl target = new UserServiceImpl();
        UserService proxy = (UserService) Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new MyInvocationHandler(target));
        proxy.save();
    }

}

```



## AOP本质

AOP 本质：

> 底层就是动态代理。

Spring AOP 内部：

- 有接口 → 用 JDK 动态代理
- 没接口 → 用 CGLIB

<br/>

spring 在bean实例化时，发现有AOP代理逻辑，返回了代理对象

初始化完成后将 代理对象放入一级缓存中。

你调用时实际调用的是代理对象



## spring事务与AOP

@Transactional  事务 就是使用AOP来实现的

<br/>

在没有AOP之前,我们要进行事务就要写一堆与业务无关的代码：

```java
public void transferMoney() {
    Connection conn = null;
    try {
        conn = getConnection();
        // 1. 开启事务（关闭自动提交）
        conn.setAutoCommit(false); 
        
        // --- 下面才是真正的核心业务逻辑 ---
        accountDao.deduct(zhangsan, 100); // 张三扣钱
        // 假设这里突然发生异常报错了！
        accountDao.add(lisi, 100);        // 李四加钱
        // ---------------------------------
        
        // 2. 如果一切顺利，手动提交事务
        conn.commit(); 
    } catch (Exception e) {
        // 3. 如果发生异常，手动回滚事务！张三的钱退回去！
        if (conn != null) {
            conn.rollback(); 
        }
    } finally {
        // 4. 关闭连接
        closeConnection(conn); 
    }
}
```



有了AOP之后，只需要加一个@Transactional注解即可

它帮我们完成了AOP在事务上的实现

- 在前置通知中，向数据库进行连接
- 执行原方法
- 如果方法执行成功，则在返回通知中进行提交（commit）
- 如果出现异常，则会回滚



### 事务失效

 `@Transactional` 的底层是 **AOP 动态代理**

那么ve'ge同类内部方法调用，事务会失效！

```java
@Service
public class BankService {

    public void doSomething() {
        // 这里执行了一些普通逻辑...
        
        // 然后调用了本类中带有事务的方法
        this.transferMoney(); 
    }

    @Transactional
    public void transferMoney() {
        accountDao.deduct(zhangsan, 100); 
        int i = 1 / 0; // 模拟报错
        accountDao.add(lisi, 100);        
    }
}
```

这是因为：this使用的是原生的对象，而不是代理对象。而AOP则必须要经过代理对象的拦截。所以就失效了

<br/>

如何解决呢：

1. 在类内部注入自身（`@Autowired private UserService self;`），然后调用 `self.methodB()`。
2. 将方法 B 移到另一个 Service 类中

















