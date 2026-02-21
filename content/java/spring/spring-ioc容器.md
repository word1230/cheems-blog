---
date: 2026-02-21T12:11:02+08:00
draft: false
slug: 042c3364
type: posts
title: Spring Ioc容器
collections: spring
categories:
  - 编程
---
IOC，DI，bean作用域，bean生命周期，循环依赖问题
<!--more-->
## IOC 是什么
IOC是控制反转，那什么是控制反转呢？

你想啊，有反转就一定有正转

正转：程序中的对象由开发人员员自己进行new，自己进行管理

反转： 由程序自己来管理程序中的对象，这个管理程序叫bean容器，也就是：由容器来管理对象

那控制反转就是这个意思：对于对象的控制权，由开发人员交给了容器

### 为什么要控制反转呢

让程序员自己来管理不行吗？为什么要让容器管理

举个栗子：

现在你要在代码里造一辆车：

> 正常情况你会这样写： 写死一个v6发动机

```java
public class Car {

    private V6engine v6engine;

    public Car(V6engine v6engine) {

        this.v6engine = v6engine;

    }
    public void run() {

        v6engine.run();

    }
    
        public static void main(String[] args) {
        try (ConfigurableApplicationContext context = SpringApplication.run(Demo02Application.class, args)) {
            Car myCar = context.getBean(Car.class);
            myCar.run();
        }
    }
}
```

现在国家要求改电车，那你就要改动Car这个类的代码了,而且是大改：
```java
public class Car {

    private ElectricMotor electricMotor;

    public Car(ElectricMotor electricMotor) {

        this.electricMotor = electricMotor;

    }
    public void run() {

        electricMotor.run();

    }
    
        public static void main(String[] args) {
        try (ConfigurableApplicationContext context = SpringApplication.run(Demo02Application.class, args)) {
            Car myCar = context.getBean(Car.class);
            myCar.run();
        }
    }
}
```

> 如果使用控制反转

首先定义一个接口
```java
public interface Engine {
    void run();
}
```


定义实现类，用@component注解，表示将这个类交给容器进行管理
```java
@Component
public class V6engine  implements Engine{

    @Override
    public void run() {
        System.out.println("v6启动");
    }

}
```


定义Car，同样交给容器管理，
```java
@Component
public class Car {

    private Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }

    public void run() {
        engin.run();
    }
    
        public static void main(String[] args) {
        try (ConfigurableApplicationContext context = SpringApplication.run(Demo02Application.class, args)) {
            Car myCar = context.getBean(Car.class);
            myCar.run();
        }
    }

}

```

那此时要换电车怎么换呢

定义一个电动发动机,同样交给容器管理， ==去掉上面v6的那个@Component== 不然会报错的。（要解决，可以提前看下面的==问题1==） 这样就可以了
```java
@Component
public class ElectricMotor implements Engine {

    @Override
    public void run() {
        System.out.println("电动发动机启动");
    }

}
```
因为容器识别到了 容器里只有一个电发动机，所以就注入给了Car。


那么控制反转的优势就出来了：将Car 与 发动机解耦了。

控制反转还有其他好处：
- 统一配置和管理
- 方便单元测试

## 如何实现IOC呢

这么一看IOC确实不错，那如何实现IOC呢
那就是 依赖注入（DI） 

那什么是DI呢：  容器将你需要的对象给你 就是DI
举个栗子，就是上面的：将电发动机给安装到（注入到）Car里就是DI

DI有三种形式:
- 通过构造器进行注入
- 通过Setter进行注入
- 通过字段进行注入


有一个注解需要认识一下： @Autowired
这个注解是告诉spring容器，让他从 容器 中 将 对象 注入进来


还是上面的Car 的例子：
### 通过构造起注入
1. 依旧先写Engine接口（跟上面一样，这里省略了）
2. 然后定义一个V6engine实现类，交给ioc容器管理（同上）
3. 然后定义一个Car

```java
@Component
public class Car {

    private Engine engine;

	@Autowired                 
    public Car(Engine engine) {
        this.engine = engine;
    }

    public void run() {
        v6engine.run();
    }
    
        public static void main(String[] args) {
        try (ConfigurableApplicationContext context = SpringApplication.run(Demo02Application.class, args)) {
            Car myCar = context.getBean(Car.class);
            myCar.run();
        }
    }

}
```

因为这里只有一个构造方法，在spring4.13版本后，识别到只有一个构造器时，默认使用这个构造器进行注入了。 

4.13 之前要使用一个注解 @Autowired

### 通过Setter字段注入
前两步是一样的
第三步：
```java
@Component
public class Car {

    private Engine engine;

    @Autowired
    public void setV6engine(Engine engine) {
        this.engine = engine;
    }

    public void run() {
        v6engine.run();
    }
    
        public static void main(String[] args) {
        try (ConfigurableApplicationContext context = SpringApplication.run(Demo02Application.class, args)) {
            Car myCar = context.getBean(Car.class);
            myCar.run();
        }
    }

}
```

这里去掉了构造器， 使用setter方法，并在其上加了@Autowired注解

### 通过字段注入

```java
@Component
public class Car {

    @Autowired
    private Engine engine;

 
    public void run() {
        engine.run();
    }
        public static void main(String[] args) {
        try (ConfigurableApplicationContext context = SpringApplication.run(Demo02Application.class, args)) {
            Car myCar = context.getBean(Car.class);
            myCar.run();
        }
    }

}

```

你可以直接在字段上加 @Autowired 注解，来实现di

### 总结一下DI的步骤
1. 首先写一个接口
2. 写一个实现， 并用@component注解标记
3. 在需要注入的地方，使用@component + @Autowired

### 问题1：如果两个类都用@component注解了，那@Autowired注入哪个

两个类都用@component注解标记了，spring懵了，注入哪个呢？
这时就会报错。
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260221132722191.png)


如何处理这种情况？三种方法：
- @Qualifier： 指定一下我要注入哪个
- @Primary：全局指定我要用那个
- 通过变量名推断

spring 的寻找逻辑是：
1. 先根据**类型（Type）**找，找到一个直接注入。
2. 找到多个？看有没有哪个戴着 `@Primary` 王冠，有就注入。
3. 没有王冠？看你有没有用 `@Qualifier` 叫名字，有就按名字找。
4. 没叫名字？最后看看你的**变量名**是不是刚好碰上了 Bean 的名字。
5. 都匹配不上？直接报错罢工。

#### @Qualifier

1. 先写接口
```java
public interface Engine {

   void run();
}

```

2. 定义两个实现类
```java
@Component
public class V6engine  implements Engine{

    @Override
    public void run() {
        System.out.println("v6启动");
    }

}

```

```java

@Component
public class ElectricMotor implements Engine {

    @Override
    public void run() {
        System.out.println("电动发动机启动");
    }

}
```

3. car
```java
@Component
public class Car {

    private Engine engine;

    @Autowired
    public Car(@Qualifier("V6engine") Engine engine){
        this.engine = engine;
    }

 
    public void run() {
        engine.run();
    }

    public static void main(String[] args) {
        try (ConfigurableApplicationContext context = SpringApplication.run(Demo02Application.class, args)) {
            Car myCar = context.getBean(Car.class);
            myCar.run();
        }
    }
}

```

这里在构造函数上用了 @qualifier。 你也可以在setter方法上 ， 或者字段上使用
同样@Autowired 可以省略


#### @Primary
1. 先写接口
```java
public interface Engine {

   void run();
}

```

2. 定义两个实现类
```java
@Component
public class V6engine  implements Engine{

    @Override
    public void run() {
        System.out.println("v6启动");
    }

}

```

```java

@Component
@Primary
public class ElectricMotor implements Engine {

    @Override
    public void run() {
        System.out.println("电动发动机启动");
    }

} 
```

这里给 ElectricMotor 加了一个 @Primary

Car 类不变， 这样所有的Engine的注入都是ElectricMotor

#### 通过字段推断
前两个步骤依旧：定义接口，定义实现类

第三步：
```java
@Component
public class Car {

    private Engine engine;


    public Car(Engine v6engine){
        this.engine = v6engine;
    }

 
    public void run() {
        engine.run();
    }

    public static void main(String[] args) {
        try (ConfigurableApplicationContext context = SpringApplication.run(Demo02Application.class, args)) {
            Car myCar = context.getBean(Car.class);
            myCar.run();
        }
    }
}
```

这里将 构造器里的 参数的名称改掉了，改成了v6engine ，spring容器通过名称推断出是v6engine。所以注入了v6engine

同样你也可以改 字段/setter里的参数都可以实现


### 问题2 引入别人的包后，也希望使用ioc管理对方的类，但是不能动对方的源码，要如何做呢？

使用 `@Configuration` 和 `@Bean`

举个例子：

依然是之前的Car ，现在有一家公司推出了一款发动机（CheemsEngine），那我希望让他被spring管理，我们之前是使用@component注解，但是人家的代码是写好的，不能改，怎么办呢。


1. 写一个配置类

```java
public class CheemsEngine implements Engine {

    @Override
    public void run() {
       System.out.println("Cheems 发动机 启动");
    }

}

```

```java
@Configuration
public class CarConfig {

    @Bean
    Engine cheemsEngine(){
        return new CheemsEngine();
    }

}
```

写一个类，用@Configration 注解标注， 然后写一个方法，用@Bean标注，表示：告诉spring，
将这个方法的返回值作为一个对象，加入到spring容器中去。

这个方法的返回值为 接口， 方法名为对象的名称，内部new 一个要让spring容器管理的对象

## Bean的作用域

既然对象被容器管理了， 那容器里是 一个对象， 还是多个对象呢
这就是Bean的作用域

作用域有几种：
1. 单例
2. 多例
3. 针对web 的 request session等

### 单例
容器中的一个类，就只有一个对象，你在什么地方去注入，都是这个对象

好处是省内存，性能好，全局唯一

spring==默认使用的就是单例==
为什么呢： 就是为了提高性能

写法：
```java
@Component
@Scope("singleton")
public class ElectricMotor implements Engine {

    @Override
    public void run() {
        System.out.println("电动发动机启动");
    }

}
```

就是这个 @Scope("singleton")  但是因为是默认的，所以一般不会写出来

### 多例
容器中，一个类有多个对象。 每次注入都是一个全新的对象

这种主要是为了解决： 你的类有一个变量是 状态值，它会随着业务变化，那张三拿到这个值，修改了， 李四再拿到这个值就不对了，它被张三修改了。 因此只能是每个人拿到的对象都是一个全新的对象。


## Bean的完整生命周期

1.  实例化












## 如何解决循环依赖问题

