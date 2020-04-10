---
title: 注解
date: 2018-12-17 18:33:33
categories: java
tags:
    - java注解
---

&emsp;&emsp;Java注解又称Java标注，是Java语言5.0版本开始支持加入源代码的特殊语法元数据。Java语言中的类、方法、变量、参数和包等都可以被标注。和Javadoc不同，Java标注可以通过反射获取标注内容。在编译器生成类文件时，标注可以被嵌入到字节码中。Java虚拟机可以保留标注内容，在运行时可以获取到标注内容。 当然它也支持自定义Java标注。

<!-- more -->

### 基本语法

- @Test注解定义

```java
public @interface Ferry {

}
```
&emsp;&emsp;使用`@interface`来声明Test是一个注解,和声明类,接口的方法是一样的,请记住,注解是不支持继承的.

- 使用

```java
@Ferry
public class DemoMain {
    @Ferry
    private static String attribute;

    @Ferry
    public static String annotation(){
        return "haha";
    }

    public static void main(String[] args) {
        System.out.println(attribute);
        System.out.println(DemoMain.annotation());
    }
}
```

![运行结果](/annotation/annotation-ferry-1.png)

&emsp;&emsp;自定义的这个注解已经可以使用了,这个没有任何限的注解可以使用在类,私有属性以及方法上,当然,因为这个注解并没有实现任何功能,所以也毫无意义.

&emsp;&emsp;当我们想要对自定义注解做一些限制时,例如限制注解的使用范围,限制注解生命周期,可以使用java提供的元注解.

### 元注解

&emsp;&emsp;元注解就是标记其他注解的注解;

- `@Target`注解:限制注解的使用范围

```java
@Target(ElementType.METHOD)
public @interface Ferry {

}
```

- `ElementType`枚举类

```java
public enum ElementType {
    /**标明该注解可以用于类、接口（包括注解类型）或enum声明*/
    TYPE,

    /** 标明该注解可以用于字段(域)声明，包括enum实例 */
    FIELD,

    /** 标明该注解可以用于方法声明 */
    METHOD,

    /** 标明该注解可以用于参数声明 */
    PARAMETER,

    /** 标明注解可以用于构造函数声明 */
    CONSTRUCTOR,

    /** 标明注解可以用于局部变量声明 */
    LOCAL_VARIABLE,

    /** 标明注解可以用于注解声明(应用于另一个注解上)*/
    ANNOTATION_TYPE,

    /** 标明注解可以用于包声明 */
    PACKAGE,

    /**
     * 标明注解可以用于类型参数声明（1.8新加入）
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * 类型使用声明（1.8新加入)
     * @since 1.8
     */
    TYPE_USE
}
```

&emsp;&emsp;可以使用`@Target(value = {ElementType.METHOD,ElementType.FIELD})`方式来指定多个值.

- `@Retention`注解:约束注解的生命周期

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Ferry {

}
```

- `RetentionPolicy`枚举类

```java
public enum RetentionPolicy {
    /**
     * 源码级别
     * 注解将被编译器丢弃（该类型的注解信息只会保留在源码里，源码经过编译后，注解信息会被丢弃，不会保留在编译好的class文件里）
     */
    SOURCE,

    /**
     * 类文件级别
     * 注解在class文件中可用，但会被JVM丢弃（该类型的注解信息会保留在源码里和class文件里，在执行的时候，不会加载到虚拟机中），请注意，当注解未定义Retention值时，默认值是CLASS，如Java内置注解，@Override、@Deprecated、@SuppressWarnning等
     */
    CLASS,

    /**
     * 运行时级别
     * 注解信息将在运行期(JVM)也保留，因此可以通过反射机制读取注解的信息（源码、class文件和执行的时候都有注解的信息），如SpringMvc中的@Controller、@Autowired、@RequestMapping等。
     */
    RUNTIME
}
```

- `@Documented`注解:被修饰的注解会生成到javadoc中

```java
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Ferry {
}
```

- `@Inherited`注解:标注的注解可以被 使用这个注解的类的子类继承

```java
@Inherited
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface X {}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface Y {}

@X
class A {}

class B extends A {}

@Y
class C {}

class D extends C {}

@Test
public void skpcfsojv(){
    System.out.println("使用了@Inherited注解: "+B.class.isAnnotationPresent(X.class));
    System.out.println("未使用@Inherited注解: "+D.class.isAnnotationPresent(Y.class));
}
```

![运行结果](/annotation/annotation-Inherited.png)

&emsp;&emsp;换句话说,就是子类可以继承父类上的使用`@Inherited`标注的注解.通过上面的例子而言,就是注解X可以被使用了X注解的类A的子类B继承.

- `@Repeatable`注解:使标注的注解可以在相同位置重复使用

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface Paths{
    Path[] value();
}


@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Repeatable(Paths.class)
@interface Path{
    String value();
}

@Path("ferry/downloads")
@Path("ferry/music")
class AA{
    
}

@Test
public void dsbucn(){
    Path[] ps = AA.class.getAnnotationsByType(Path.class);
    for (Path p : ps) {
        System.out.println(p.value());
    }
}
```

![运行结果](/annotation/annotation-repeatable.png)

&emsp;&emsp;在有`@Repeatable`注解之前,我们想实现这样的功能就只能在注解中定义数组元素来实现.`@Repeatable`注解的value元素赋值必须是一个注解的class类型,而这个注解中的value元素的数据类型必须是你定义的注解的数组类型,其实作用也就是和你直接定义一个数组来接受差不多.在获取注解中的元素的值时,针对这种被`@Repeatable`注解标注的注解而言,我们需要使用`getAnnotationsByType()`方法获取注解的数组类型然后循环获取值.

### 注解元素

&emsp;&emsp;当注解中不含有任何成员时,这个注解被称之为标记注解,如上的自定义注解就是一个标记注解,它的唯一目的就是标记声明.

- 有成员的注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Ferry {
    String name() default "default";
}
```

&emsp;&emsp;注解中的元素的声明采用了方法的声明方式,后面使用default关键字来声明默认值,当然,默认值不是必须的,但是,没有设置默认值name在使用这个注解的时候这个成员就必须赋值,否则会报编译错误.

- 使用注解并进行测试

```java
@Ferry(name = "love")
class User{
}
@Ferry
class Employee{
}

@Test
public void hjsvbj(){
    Ferry ferry = User.class.getAnnotation(Ferry.class);
    System.out.println(ferry.name());
    Ferry f = Employee.class.getAnnotation(Ferry.class);
    System.out.println(f.name());
    System.out.println(ferry.equals(f));
}
```

![运行结果](/annotation/annotation-ferry-2.png)

&emsp;&emsp;通过`Class`类的`getAnnotation()`方法获取到在这个类上使用的注解的实例,然后将注解的元素name输出.可以看到,一个输出的是指定的name值,一个输出的是默认值.显然,注解用在不同的地方会构建出不同的实例,另一个有力证据就是使用`equals()`方法判断输出false.

### 注解元素的数据类型

1. 基本数据类型(byte,char,short,int,float,long,double)

2. String

3. Class

4. enum

5. Annotation

6. 上述类型的数组

&emsp;&emsp;需要注意的是,注解元素的数据类型不能使用包装类替代基本数据类型,凡是使用不合法的数据类型都会报编译错误.

- 示例

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Ferry {
    String name() default "";

    int amount() default -1;

    Class showClass() default Object.class;

    Sex showSex() default Sex.MALE;

    Honey love() default @Honey(mySex = Sex.FEMALE);

    int[] amounts() default {};

    enum Sex {
        MALE,FEMALE;
    }

    @interface Honey {
        Sex mySex() default Sex.FEMALE;
    }
}
```

```java
@Test
public void dscslkndv(){
    Ferry ferry = User.class.getAnnotation(Ferry.class);
    System.out.println(ferry.toString());
}
```

![运行结果](/annotation/annotation-ferry-3.png)

- 特殊的元素`value`

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Look{
    String value();
}
@Look("see")
@Test
public void cbsiflk(){
    try {
        Look look = this.getClass().getMethod("cbsiflk").getAnnotation(Look.class);
        System.out.println(look);
    } catch (NoSuchMethodException e) {
        e.printStackTrace();
    }
}
```

&emsp;&emsp;当注解中有元素的名称为`value`时,在使用这个注解时为value元素赋值可以不使用key=value的形式而直接在括号中赋值,编译器会默认将该值赋予注解的value元素,当然,赋值的数据类型必须和注解中value元素的数据类型一致,否则会报编译错误.

### java内置注解

- `@Override`:用以注明此方法重写了父类的方法.

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

- `@Deprecated`:用以注明此类或者方法已经过时,不推荐使用.

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```

- `@SuppressWarnnings`:用以关闭编译器对类、方法、成员变量、变量初始化的警告.

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```

&emsp;&emsp;数组接收的值:

1. deprecation：使用了不赞成使用的类或方法时的警告；

2. unchecked：执行了未检查的转换时的警告，例如当使用集合时没有用泛型 (Generics) 来指定集合保存的类型; 

3. fallthrough：当 Switch 程序块直接通往下一种情况而没有 Break 时的警告;

4. path：在类路径、源文件路径等中有不存在的路径时的警告; 

5. serial：当在可序列化的类上缺少 serialVersionUID 定义时的警告; 

6. finally：任何 finally 子句不能正常完成时的警告; 

7. all：关于以上所有情况的警告。
