---
title: 使用javaconfig配置spring项目
date: 2018-12-17 18:46:57
tags:
    - spring
    - javaconfig
    - 自动装配
    - 高级装配
categories: java
---

&emsp;&emsp;使用javaconfig对spring项目进行配置，记录一下常用的注解及其含义，以方便后期回顾。此项目是使用maven搭建的非web项目，是我学习spring练手用的，是向web项目的过渡阶段产物。

<!-- more -->

### Maven依赖

* 先添加spring需要的依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.0.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.0.6.RELEASE</version>
    <scope>test</scope>
</dependency>
<!--测试依赖-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
<!--日志依赖-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
<!--lombok-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.persistence</groupId>
    <artifactId>persistence-api</artifactId>
    <version>1.0</version>
</dependency>
```

### 自动化装配bean

&emsp;&emsp;新建一个javaconfig包，所有的配置类都放在这个包下面。

#### @Configuration 配置类声明

* 新建一个类，这里我的类名为`BaseBeanJavaConfig`

```java
@Configuration
public class BaseBeanJavaConfig {
}
```

&emsp;&emsp;`@Configuration`注解声明这是一个配置类,所有的配置类都必须使用这个注解，否则spring又怎么会知道他是一个配置类呢。

#### @ComponentScan 组件扫描

&emsp;&emsp;组件扫描有多种方式可供选择：

1.没有使用`@ComponentScan`注解，那么将按照默认规则扫描该配置类所在包。

2.使用`@ComponentScan`注解，形如下面代码块所示，那么spring将扫描`service.impl`包及向下所有包中含有`@Component`注解的类,在{}中可以使用`,`分隔设置多个包，这样可以扫描多个包啦。

```java
@Configuration
@ComponentScan(basePackages = {"cn.org.ferry.sys.service.impl"})
public class BaseBeanJavaConfig {
}
```

3.使用class类型，这种方式spring将扫描该类所在包及向下所有包，当然，这种方式也可以用`,`分隔设置多个类。

```java
@Configuration
@ComponentScan(basePackageClasses = {SysUserServiceImpl.class})
public class BaseBeanJavaConfig {
}
```

&emsp;&emsp;一般情况下，我们都是将配置类单独放在一个包中的，毕竟如果业务代码包中多了一个配置类感觉上也怪怪的，所以第一种方式放弃，第二种和第三种方式相比较，如果你重构了代码，包路径改变了，那么使用第二种方式就会出现问题，所以一般情况下采取第三种方案来实现组件扫描。

&emsp;&emsp;值得一提的是，不管使用那种方式扫描包，你需要装配的bean都需要在类级别上加上注解，毕竟spring扫描的时候也只是扫描指定的包下面含有该注解的类。

```java
// 说一下我对这四种注解的简单看法，目前的理解就是在不同的类中使用特定的注解让人更容易理解这个类的作用
@Component  // 不管什么情况，用它都不会报错
// @Controller 控制器中使用该注解
// @Service service.impl中使用该注解
// @Repository  与mybatis框架整合时 mapper 包中使用此注解
public class Student {
}
```

#### 装配bean

&emsp;&emsp;以上操作即可通过spring生成bean，那么接下来就是装配bean了，简单说就是如何使用这个bean。

* 就拿上面那个`Student`类来举例，添加一个方法。

```java
@Component
public class Student {
    public String toString(){
        return this.getClass().getSimpleName();
    }
}
```

* 在测试类中进行测试这个类有没有实现自动装配

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = BaseBeanJavaConfig.class)
public class StudentTest {
    @Autowired
    private Student student;
    @Test
    public void test(){
        System.out.println(student.toString());
    }
}
```

![测试结果](/spring-javaconfig/spring_auto_bean_test.png)

&emsp;&emsp;当然bean的使用不局限于设置属性这一种方法，也可以通过构造器或者是任意一个方法的形参列表传递。下面的代码结果与上述图片展示结果一样。

```java
public class StudentUse {
    private Student student;
    @Autowired
    public StudentUse(Student student){
        this.student = student;
    }
    public void info(){
        System.out.println(student.toString()); // 使用这个bean
    }
}
```

&emsp;&emsp;通过给`@Autowired`注解加上属性`required = false`来避免当没有匹配的bean时spring抛出异常，但是这样设置就得在使用这个bean时进行判空了，因为它有可能没有匹配的bean而是个空值。

### 显式装配bean

&emsp;&emsp;有时候，自动化配置方案行不通，就只能够显式的配置，例如，你希望将第三方类库中的类配置为bean，但是你没有办法对该类进行修改添加`@Component`注解,那么可以采用显式装配方式进行配置。

#### 简单bean配置

&emsp;&emsp;基于一些因素，现在有一个Apple类无法通过自动装配完成bean的装配，现在需要明确的配置该bean。

- Apple类样例

```java
public class Apple {
    public String toString(){
        return this.getClass().getSimpleName();
    }
}
```

- 配置

```java
@Configuration
public class RootJavaConfig {
    @Bean
    public Apple apple(){
        return new Apple();
    }
}
```

&emsp;&emsp;通过上面代码，在一个配置类中的方法上使用`@Bean`注解，返回值类型是你希望注册的Bean的类型，在该方法中，你可以使用一切java所能实现的技术来满足你的需求以便返回实例。值得一提的是，你可以在该方法的形参列表中传递其他的bean来配合你完成工作。

- 测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = RootJavaConfig.class)
public class AppleTest {
    @Autowired
    private Apple apple;
    @Test
    public void test(){
        System.out.println(apple.toString());
    }
}
```

![测试结果](/spring-javaconfig/spring_bean_test2.png)

&emsp;&emsp;显而易见，这样装配是没有问题的，在实际应用中，我们可能会使用到第三方类库中的类，例如数据源，那么我们就可以使用这种显式装配的方式将他配置起来。

#### 给bean设置id

&emsp;&emsp;在默认的情况下，bean的id是和带有`@Bean`注解的方法名一样的，如上例，bean的id为apple。在实际应用中，我们有可能需要自定义bean的id，那么可以通过下面方式完成操作。

- 设置bean的id

```java
@Configuration
public class RootJavaConfig {
    @Bean(name = "appleBean")
    public Apple apple(){
        return new Apple();
    }
}
```

&emsp;&emsp;通过在`@Bean`注解上添加`name`属性值为bean赋上id。

### 混合配置

&emsp;&emsp;项目的配置需要视情况而定，有时候使用xml配置项目是最佳方案，而在spring中，xml配置和javaconfig配置并不是互斥的，那么就可以同时使用这两种方案来实现项目的最优配置。

&emsp;&emsp;在自动装配过程中，spring并不会在意你使用的bean是怎么来的，如果你引用了一个外部的配置中的bean，那么你只需要引用一下那个配置类/配置文件即可。

#### 配置类组合

&emsp;&emsp;在实际应用中，我们可能根据功能的区分定义多个配置类，这样便于维护以及理解，使得配置类不会过于臃肿。那么问题来了，如何整合众多的配置类呢？

- 配置类

```java
@Configuration
@ComponentScan(basePackageClasses = {Student.class})
public class BaseBeanConfig {
    @Bean(name = "sysUserServiceImpl")
    public ISysUserService sysUserServiceImpl(){
        return new SysUserServiceImpl();
    }
}

@Configuration
public class DataDourceConfig {
}
```

- 整合配置类

```java
@Configuration
@Import({BaseBeanConfig.class,DataDourceConfig.class})
public class RootJavaConfig {
}
```

&emsp;&emsp;使用`@Import`注解引入一个配置类，在括号中填入需要引入的配置类的class类型，更通用的方法是建立一个根配置类，在这个配置类中引入所有的配置类。

#### 配置类引入xml配置文件

- 引入配置文件

```java
@Configuration
@Import({BaseBeanConfig.class})
@ImportResource({"classpath:/spring/applicationContext-spring.xml"})
public class RootJavaConfig {
}
```

&emsp;&emsp;使用`@ImportResource`注解，后面跟上字符串类型的配置文件的相对路径即可引入对应的配置文件，同样的，也可以使用`,`分隔然后引入多个配置文件。

### 高级装配

&emsp;&emsp;spring的bean装配技术还有许多技巧，借助它们可以实现更为高级的bean装配功能.

#### profile

&emsp;&emsp;在实际开发中，我们在项目开发到一定程度时需要迁移环境，例如开发环境，测试环境，生产环境等之间的切换。在不同的环境可能需要创建不同的bean，那么我们可以借助`@Profile`注解来实现这个功能。`@Profile`注解有两种使用方式，下面列举。

- 例子

```java
public interface Fruits {
    String info();
}
public class Banana implements Fruits {
    public String info() {
        return this.getClass().getSimpleName();
    }
}
public class Apple implements Fruits {
     public String info() {
         return this.getClass().getSimpleName();
     }
 }
```

- 类级别使用`@Profile`注解

```java
@Configuration
@ComponentScan(basePackageClasses = {Fruits.class})
@Profile("dev")
public class BaseBeanConfig {
    @Bean("apple")
    public Fruits apple(){
        return new Apple();
    }
}
```

- 测试，在测试类中使用`@ActiveProfiles`注解来选择激活的环境。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = RootJavaConfig.class)
@ActiveProfiles("dev")
public class FruitTest {
    @Autowired
    private Fruits fruits;
    @Test
    public void test(){
        System.out.println(fruits.info());
    }
}
```

- 方法级别使用`@Profile`注解

```java
@Configuration
@ComponentScan(basePackageClasses = {Fruits.class})
public class BaseBeanConfig {
    @Bean("apple")
    @Profile("dev")
    public Fruits apple(){
        return new Apple();
    }
    @Bean("banana")
    @Profile("test")
    public Fruits banana(){
        return new Banana();
    }
}
```

- 测试，此时激活dev环境，那么控制台会打印出Apple

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = RootJavaConfig.class)
@ActiveProfiles("dev")
public class FruitTest {
    @Autowired
    private Fruits fruits;
    @Test
    public void test(){
        System.out.println(fruits.info());
    }
}
```

![测试结果](/spring-javaconfig/spring_bean_test2.png)

#### 激活profile

&emsp;&emsp;spring在确定哪个profile处于激活状态时，依赖于两个属性：`spring.profiles.active`和`spring.profiles.default`（`spring.profiles.active`的优先级更高一点）。设置这两个属性值有下列几种办法。

- 作为`DispatcherServlet`的初始化参数；
- 作为Web应用的上下文参数；
- 作为JDNI条目；
- 作为环境变量；
- 作为JVM的系统属性；
- 集成测试类中，使用`@ActiveProfiles`注解设置。

#### 处理自动装配的歧义性

&emsp;&emsp;在使用自动装配的时候，可能一个接口下面有很多实现类，我们为此都实现了`@Component`注解，那么在自动装配的时候，spring就会无法判断到底要选择哪个实现类进行装配。

- `@Primary`注解，首选标记

&emsp;&emsp;可以在你希望装配的实现类上除了`@Component`注解再加上`@Primary`注解，这表明当接口下有很多实现类都实现了`@Component`注解时将首选使用了`@Primary`注解的实现类进行装配。

- `@Qualifier`注解，使用限定符来限定注入的bean的具体实现

&emsp;&emsp;也可以不使用`@Primary`注解，那么在注入bean的地方可以使用`@Qualifier`注解表明本次注入的bean。

```java
    @Autowired
    @Qualifier("apple")
    private Fruits fruits;
```

- 限定符默认为这个bean的ID，也可以自定义限定符

```java
@Component
@Qualifier("apple")
public class Apple implements Fruits {
    public String info() {
        return this.getClass().getSimpleName();
    }
}
```
