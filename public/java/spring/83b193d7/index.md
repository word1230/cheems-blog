# AOP入门


 aop概念,使用

<!--more-->

## 什么是AOP

AOP即 面向切面编程( Aspect-Oriented Programming).

它的作用是将那些与业务逻辑无关,但要被业务模块共同调用的代码(如日志,鉴权等)封装起来,统一进行调用

## 为什么要使用AOP

AOP的好处：

1. 减少重复代码（所有日志、鉴权代码统一管理）
2. 降低耦合度（将业务代码与日志、鉴权等非业务代码分开）



举个栗子

有一个Car 类

```java

@Component
public class Car {
    public void run() {
        System.out.println("启动");
    }
}

```

现在要求在 启动（run方法）前 进行检查并输出日志

<br/>

常规的做法：  直接在run 里面加入日志输出

```java
@Slf4j
@Component
public class Car {
    public void run() {
         log.info("启动前检查");
        System.out.println("启动");
    }
}

```

但是如果还有 飞机， 船，公交车等 都要进行， 那就要在每种交通工具的run方法里加入日志

这样重复代码就多了，而且在写run（业务逻辑）时还要考虑日志，耦合度高

如果哪天日志逻辑改了，那就改很多地方

<br/>

使用AOP来做

```java
@Aspect
@Component
public class demoAspect {

    @Before("execution(* com.example.demo02..*.run(..))")
    public void beforeRun(){
        System.out.println("[AOP 介入]，车辆启动前检查");
    }

    @After("execution(* com.example.demo02..*.run(..))")
    public void afterRun(){
        System.out.println("[AOP 介入],车辆启动后检查");
    }

}
```

通过这样一个类，就能实现对所有交通工具 进行 日志添加。

你在Car（业务类）里只需要处理业务逻辑

## 如何使用AOP呢

### 核心概念

- 切面: 要增强的功能,被封装起来与业务无关的功能就是切面,比如日志。 他是一个类， 例如上面的demoAspect
- 切点: 一个规则表达式，表示通知要应用到哪些类上去
- 通知: 一个方法，告诉程序在什么时候做什么事
  - 前置通知: 在执行业务代码前
  - 后置通知: 在执行业务代码后,无论是否发生异常都会执行
  - 异常通知: 在执行业务代码后出现异常,需要做的操作,比如事务回滚
  - 返回通知: 在执行业务代码无误后会执行的操作
  - 环绕通知: 可以自行控制在业务代码前/后

<br/>

切点表达式: 访问修饰符 返回值 包名.包名.包名...类名.方法名(参数列表)

访问修饰符可以省略 ， 可以使用 * 来代表任意  可以使用 `..` 表示有无参数均可

例如： `* com.example.*.run(..)`



### 实战

引入依赖

```xml
	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
			<version>3.5.10</version>
			<scope>compile</scope>
		</dependency>

```



启用注解

```java
@EnableAspectJAutoProxy
@SpringBootApplication
public class Demo02Application {

	public static void main(String[] args) {
		SpringApplication.run(Demo02Application.class, args);
	}

}
```



先写一个类

```java
@Component
public class Car {
    public void run() {
        System.out.println("真正的汽车正在行驶...");
    }
}
```

写一个切面类



```java

@Aspect  //告诉spring 这是一个切面类
@Component 
public class demoAspect {

    //定义一个切点
    @Pointcut("execution(* com.example.demo02..*.run(..))")
    public void runPointcut() {
    }

    //定义前置通知
    @Before("runPointcut()")
    public void beforeRun(){
        System.out.println("[AOP 介入]，车辆启动前检查");
    }

    //定义后置通知
    @After("runPointcut()")
    public void afterRun(){
        System.out.println("[AOP 介入],车辆启动后检查");
    }

    //返回通知
    @AfterReturning("runPointcut()")
    public void afterReturningRun() {
        System.out.println("[AOP 介入],方法返回通知");
    }

    //异常通知
    @AfterThrowing(pointcut = "runPointcut()", throwing = "exception")
    public void afterThrowingRun(Exception exception) {
        System.out.println("[AOP 介入],异常通知: " + exception.getMessage());
    }

    //环绕通知
    @Around("runPointcut()")
    public Object aroundRun(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("[AOP 介入],环绕通知-前");
        try {
            //执行原方法
            Object result = joinPoint.proceed();
            System.out.println("[AOP 介入],环绕通知-后");
            return result;
        } catch (Throwable throwable) {
            System.out.println("[AOP 介入],环绕通知-异常");
            throw throwable;
        }
    }

}

```

<br/>



当然也可以 直将 通知和切点写到一起

```java
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect      // 告诉 Spring：我是一个切面类（交警局/记录仪）
@Component   // 切面本身也要交给 Spring 容器管理
public class SmartDashcamAspect {

    // @Before 这是一个“前置通知”
    // execution(...) 就是“切点规则”：意思是拦截 com.example 包下，任何类里的 run 方法！
    @Before("execution(* com.example.*.run(..))")
    public void beforeRun() {
        System.out.println("【AOP 介入】车辆启动前：系统正在校验驾驶员身份...");
    }

    // @After 这是一个“后置通知”
    @After("execution(* com.example.*.run(..))")
    public void afterRun() {
        System.out.println("【AOP 介入】车辆熄火后：已将本次行驶轨迹同步至云端。");
    }
}
```



---

> 作者: cheems  
> URL: http://localhost:1313/java/spring/83b193d7/  

