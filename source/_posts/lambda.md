---
title: lambda
date: 2019-08-18 10:23:11
tags:
    - lambda
categories: java
---

&emsp;&emsp;`lambda表达式`是一个可传递的代码块。

<!-- more -->

### 语法

- 基本语法：参数，箭头（->）以及一个表达式或者代码块。

```java
(String msg, String str) -> return msg.length()>str.length()
```

```java
(String msg, String str) -> {
    if(msg.length()>str.length() > 0) return 1;
    else if(msg.length()>str.length() == 0) return 0;
    else return -1;
}
```

&emsp;&emsp;*如果代码块在有些分支有返回值，有些分支没有返回值是不合法的。返回值的类型无需指定，lambda表达式的返回值类型总是有上下文推导而出。lambda表达式可以捕获外围作用域中变量的值，但是所捕获的变量必须实际上是最终变量*

- 无参表达式：提供空括号

```java
() -> System.out.println("no arguments lambda")
```

- 忽略参数类型：若参数类型可推导，可忽略参数类型。若只有一个参数且参数类型可推导，可省略括号。

```java
(msg, str) -> return msg.length() > str.length()
```

```java
msg -> System.out.println(msg)
```

### 函数式接口

- 定义：只有一个抽象方法的接口。自己设计接口时，如果接口只有一个抽象方法，那么可以使用注解`@FunctionalInterface`标记这个接口。这样，当你向这个接口再次添加一个抽象方法时，编译器会产生一个错误。另外javadoc页会指出此接口为函数式接口。

- demo：返回指定字符串10倍的长度

```java
/**
* 函数式接口
*/
@FunctionalInterface
public interface LengthInterface{
    int renderLength(String msg);
}

/**
* 匿名内部类方式实现
*/
@Test
public void innerClass(){
    System.out.println(new LengthInterface() {
        @Override
        public int renderLength(String msg) {
            return msg.length()*10;
        }
    }.renderLength("ferry"));
}

/**
* lambda表达式方式实现
*/
@Test
public void lambda(){
    LengthInterface lengthInterface = msg -> msg.length()*10;
    System.out.println(lengthInterface.renderLength("ferry"));
}
```

![初始化git项目方式2](/lambda/lambda_functionalInterface.png)

&emsp;&emsp;定义一个函数式接口，这个接口只有一个抽象方法，这个抽象方法的用途是希望获取到指定的字符串进过渲染后的长度。在lambda表达式出现之前，我们想要实现这样的功能，最好的做法是通过匿名内部类来实现，但是在这里，我们可以让函数式接口的引用指向一个lambda表达式，通过lambda表达式定义渲染长度规则，然后在通过引用调用函数式接口的抽象方法。很显然，虽然内部类和lambda都可以实现功能，但是不管是从代码的简洁度，可读性还是从性能对比上（内部类的类加载，解析，初始化等高于lambda表达式的性能开销）lambda表达式都是我们的首选方案。

- 常用函数式接口

函数式接口              | 参数类型 | 返回类型 | 抽象方法 | 描述                      | 其他方法
--------------------- | ------- | ------- | ------- | ------------------------- | ----------------------------
Runnable              | 无      | void    | run     | 作为无参数或返回值的动作运行  |
Supplier<T>           | 无      | T       | get     | 提供一个T类型的值           |
Consumer<T>           | T       | void    | accept  | 处理一个T类铟的值           | andThen
BiConsumer<T,U>      | T,U     | void    | accept  | 处理T和U类型的值            | andThen
Function<T,R>        | T       | R       | apply   | 有一个T类型参数的函数        | compose, andThen, identity
BiFunction<T,U,R>   | T,U     | R       | apply   | 有T和U类型参数的函数        | andThen
UnaryOperator<T>      | T       | T       | apply   | 类型T上的一元操作符         | compose, andThen, identity
BinaryOperator<T>     | T,T     | T       | apply   | 类型T上的二元操作符         | andThen, maxBy, minBy
Predicate<T>          | T       | boolean | test    | 布尔值函数                 | and, or, negate, isEqual
BiPredicate<T,U>     | T,U     | boolean | test    | 有两个参数的布尔值函数       | and, or, negate

- 基本类型的函数式接口

函数式接口           | 参数类型 | 返回类型 | 抽象方法 
------------------- | ------- | ------- | -------------
BooleanSupplier     | none    | boolean | getAsBoolean
PSupplier           | none    | p       | getAsP
PConsumer           | p       | void    | accept
ObjPConsumer< T>    | T,p     | void    | accept
PFunction<T>        | p       | T       | apply
PToQFunction        | p       | q       | applyAsQ
ToPFunction<T>      | T       | p       | applyAsP
ToPBiFunction<T,U>  | T,U     | p       | applyAsP
PUnaryOperator      | p       | p       | testAsP
PBinaryOperator     | p,p     | p       | testAsP
PPredicate          | p       | boolean | test

*注：p,q为int, long, double;P, Q为Integer, Long, Double*

### 方法引用

&emsp;&emsp;将现有的方法传递给函数式接口，等价于lambda表达式

```java
String::length  <==> msg -> msg.length()
(String msg) -> System.out.println(msg)  <==> System.out::println
```

&emsp;&emsp;语法：用操作符`::`分割对象名或类名与方法名，有三种用法，第一：对象::非静态方法名；第二：类名::静态方法名；第三：类名::非静态方法名。前两种方式，方法引用等价于提供方法参数的lambda表达式。第三种情况，第一个参数将成为方法的目标，例如，(msg, str) -> msg.equals(str) 等同于 String::equals。

### 构造器引用

&emsp;&emsp;和方法引用很类似，只不过方法名为new，具体方式为：类名::new，例如：String::new，具体使用类的哪一个构造器取决于上下文。数组的构造器引用：String[]::new，它有一个参数，即数组的长度，等价于length -> new String[length]。


