---
title: 泛型
date: 2018-12-17 18:39:38
tags:
    - 泛型
categories: java
---

&emsp;&emsp;Java 泛型（generics）是 JDK 5 中引入的一个新特性, 泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。

<!-- more -->

### 泛型类

#### 泛型初涉

- 定义泛型类

```java
public class Point<T> {
    private T x;
    private T y;

    public Point(){}

    public Point(T x, T y){
        this.x = x;
        this.y = y;
    }

    public T getX() {
        return x;
    }

    public void setX(T x) {
        this.x = x;
    }

    public T getY() {
        return y;
    }

    public void setY(T y) {
        this.y = y;
    }

    @Override
    public String toString() {
        return "Point{" +
                "x=" + x +
                ", y=" + y +
                '}';
    }
}
```

&emsp;&emsp;在类名`Point`后面加上一对尖括号,尖括号中一个英文字母,这样就表示这个类是一个泛型类.这个英文字母在实际应用中可以自己替换成任意一个英文字母,但是在该类中使用这个泛型的时候字母要和这个尖括号中的字母保持一致,不然会报编译错误.在这个泛型类中使用这个T,可以将T当做一个类来进行使用,例如当做Object类,那么T可以当做私有属性的类型,可以作为构造器的形参类型,可以作为方法的形参类型或者是返回值类型来使用.

- 实例化泛型类

```java
@Test
public void genericityClassTest(){
    Point<Integer> point = new Point<Integer>(10,10);
    Point<Float> p = new Point<Float>(10f,10f);
    System.out.println(point);
    System.out.println(p);
}
```

![运行结果](/genericity/genericity_class.png)

&emsp;&emsp;在实例化泛型类的时候,需要在尖括号中传递一个类名,这个类名就是你希望限定这个对象类型的类名,例如你希望这个对象都是以Integer类型来进行数据处理,那么在尖括号中传递Integer,那么这个对象所实例化的那个类中的T将全部代表Integer类,此时你使用这个泛型的方法,构造器时,如果有T作为参数类型的那么就要求你全部使用Integer类型的参数,否则会报编译时错误.

#### 多泛型变量定义

&emsp;&emsp;多泛型定义的java类形如Map集合.

- 定义

```java
/**
 * 坐标类
 * @author ferry ferry_sy@163.com
 * @param <X> x轴坐标的类型
 * @param <Y> y轴坐标的类型
 */

public class Point<X,Y> {
    private X x;
    private Y y;

    public Point(){}

    public Point(X x, Y y){
        this.x = x;
        this.y = y;
    }

    public X getX() {
        return x;
    }

    public void setX(X x) {
        this.x = x;
    }

    public Y getY() {
        return y;
    }

    public void setY(Y y) {
        this.y = y;
    }

    @Override
    public String toString() {
        return "Point{" +
                "x=" + x +
                ", y=" + y +
                '}';
    }
}
```

&emsp;&emsp;这里重构了坐标类,使用两个泛型变量进行类的定义,其中X代表x轴的数据类型,Y代表y轴数据类型.当然,在定义泛型类时你可以使用任意多的泛型变量来满足你的需求.

- 实例化

```java
@Test
public void genericityClassTest(){
    Point<Integer, Double> point = new Point<Integer, Double>(10,10D);
    System.out.println(point);
}
```

![运行结果](/genericity/genericity-class-more.png)

&emsp;&emsp;在创建实例的时候,这里选择将x轴使用Integer类型,y轴使用Double类型来实例化坐标.

#### 字母命名规范

&emsp;&emsp;在定义泛型时可以使用任意字母,他们所表示的意义是一样的,但是在实际应用中,我们还是应该使用有意义的字母来进行泛型的定义

1. E — Element，常用在java Collection里，如：List<E>,Iterator<E>,Set<E>

2. K,V — Key，Value，代表Map的键值对

3. N — Number，数字

4. T — Type，类型，如String，Integer等等

### 泛型接口

#### 接口定义

- 定义

```java
public interface Fruits<T> {
    String toString(T var);
}
```

##### 实现类方式一

- 定义

```java
@Data
public class Banana implements Fruits<String> {
    private String name;

    public Banana() {
    }

    public Banana(String name) {
        this.name = name;
    }

    public String toString(String var) {
        return name.concat(" ").concat(var);
    }
}
```

&emsp;&emsp;使用非泛型类实现泛型接口,此时需要将泛型变量赋予具体的类名,此时实现该接口方法时编译器会检测所实现的方法的泛型变量所代表的类型与实现泛型接口的数据类型是否一致,不一致时编译器会认为你并没有实现这个接口的方法.

&emsp;&emsp;`@Data`是`lombok`的注解,它的作用是用来自动生成getter和setter等方法.

- 使用

```java
@Test
public void genericityInterfaceTest1(){
    Banana banana = new Banana();
    banana.setName("banana");
    System.out.println(banana.toString("is delicious!"));
}
```

![运行结果](/genericity/genericity-interface-1.png)

#### 实现类方式二

- 定义

```java
public class Apple<T> implements Fruits<T> {
    private T weight;

    public Apple() {
    }

    public String toString(T weight) {
        this.weight = weight;
        return toString();
    }

    @Override
    public String toString() {
        return "Apple{" +
                "weight=" + weight +
                '}';
    }
}
```

&emsp;&emsp;使用泛型类实现泛型接口,这个泛型类的泛型变量可以有多个,但是必须有一个变量与泛型接口的泛型变量保持一致.

- 使用

```java
@Test
public void genericityInterfaceTest2(){
    Apple<Double> apple = new Apple<Double>();
    System.out.println(apple.toString(0.5D));
}
```

![运行结果](/genericity/genericity-interface-2.png)

### 泛型方法

- 在一个普通java类中定义一个泛型类.

```java
private <T> T genericity(T var){
    return var;
}

/**
 * 泛型方法测试
 */
@Test
public void genericityMethodTest(){
    Banana banana = new Banana("banana");
    System.out.println(genericity(banana).getClass().getName());
}
```

![运行结果](/genericity/genericity-method.png)

&emsp;&emsp;定义一个方法,在返回值类型前面加上泛型变量即可声明这是一个泛型方法,可以通过形参传递泛型,也可以返回泛型所指定的对象.在本例中实例化`Banana`类,然后传递到这个泛型方法中,最后返回这个`Banana`对象.

&emsp;&emsp;静态方法也可以定义为泛型方法,形式与上述方式类似.

### 泛型的通配符

#### 无边界通配符

```java
List<?>
```

#### 上边界通配符

```java
@Data
class A{
    protected String name;
}
@Data
class B extends A{
    private String id;
}
@Data
class C extends A{
    private String sex;
}
private List<? extends A> sbadjh(List<? extends A> list){
    list.forEach(a -> a.name = "this is a.");
    return list;
}
@Test
public void sabdkjk(){
    List<A> aList = new ArrayList<>();
    B b = new B();
    C c = new C();
    aList.add(b);
    aList.add(c);
    sbadjh(aList).forEach(a -> System.out.println(a.name));

    List<B> bList = new ArrayList<>();
    B b1 = new B();
    B b2 = new B();
    bList.add(b1);
    bList.add(b2);
    sbadjh(bList).forEach(a -> System.out.println(a.name));

    List<C> CList = new ArrayList<>();
    C c1 = new C();
    C c2 = new C();
    CList.add(c1);
    CList.add(c2);
    sbadjh(CList).forEach(a -> System.out.println(a.name));
}
```

#### 下边界通配符

&emsp;&emsp;下边界通配符使用和上边界通配符正好反过来,将`extends`关键字换成`super`.