---
title: 枚举类
date: 2018-12-17 18:38:35
categories: java
tags:
    - 枚举类型
---

&emsp;&emsp;枚举类型（Enumerated Type） 很早就出现在编程语言中，它被用来将一组类似的值包含到一种类型当中。而这种枚举类型的名称则会被定义成独一无二的类型描述符，在这一点上和常量的定义相似。不过相比较常量类型，枚举类型可以为申明的变量提供更大的取值范围。

<!-- more -->

### 定义枚举类型

- 定义性别

```java
public enum Sex {
    MALE,FEMALE;
}
```

&emsp;&emsp;枚举的定义是通过关键字`enum`定义的,它表明这个类是一个枚举类,其中,有几种类型,就直接定义就好了,中间用逗号分隔,用封号结尾,枚举值一般采用大写形式.

&emsp;&emsp;我们所有的枚举类实际上都是继承自Enum类,那么根据java的单继承特性,枚举类是不能在继承其他的类了,当然,这并不妨碍它实现接口.

- 使用

```java
@Test
public void sdmkl(){
    System.out.println("Sex.FEMALE -> "+Sex.FEMALE);
    System.out.println("Sex.FEMALE.name() -> "+Sex.FEMALE.name());
    System.out.println("Sex.FEMALE.toString() -> "+Sex.FEMALE.toString());
    System.out.println("Sex.FEMALE.ordinal() -> "+Sex.FEMALE.ordinal());
    System.out.println("Sex.valueOf(\"FEMALE\") -> "+Sex.valueOf("FEMALE"));
    System.out.println("Enum.valueOf(Sex.class,\"FEMALE\") -> "+Enum.valueOf(Sex.class,"FEMALE"));
    System.out.println("Arrays.toString(Sex.values()) -> "+Arrays.toString(Sex.values()));
}
```

![运行结果](/enum/enum-test-1.png)

&emsp;&emsp;前面三种方式输出的都是变量的字符串形式;`ordinal()`方法输出该枚举变量声明的序列,序列从0开始;`valueOf()`方法重载了两种实现方式,第一种传递枚举变量的名称,第二种传递枚举类的Class对象和名称,两者都返回对应的枚举变量名称;`values()`方法返回该枚举类的所有枚举变量数组.

### 自定义构造器

&emsp;&emsp;有时候,我们需要给自定义的枚举类型添加一些中文描述或者添加一些标志之类的操作,那么我们可以自定义构造器以满足我们的需求.

- 自定义构造器

```java
enum Sex2{
    MALE("男"),
    FEMALE("女");

    private String description;

    private Sex2(String description){
        this.description = description;
    }

    public String getDescription(){
        return description;
    }
}

@Test
public void sfsnkjc(){
    System.out.println("Sex2.FEMALE -> "+Sex2.FEMALE);
    System.out.println("Sex2.FEMALE.name() -> "+Sex2.FEMALE.name());
    System.out.println("Sex2.FEMALE.toString() -> "+Sex2.FEMALE.toString());
    System.out.println("Sex2.FEMALE.ordinal() -> "+Sex2.FEMALE.ordinal());
    System.out.println("Sex2.valueOf(\"FEMALE\") -> "+Sex2.valueOf("FEMALE"));
    System.out.println("Enum.valueOf(Sex2.class,\"FEMALE\") -> "+Enum.valueOf(Sex2.class,"FEMALE"));
    System.out.println("Arrays.toString(Sex2.values()) -> "+Arrays.toString(Sex2.values()));

    System.out.println(Sex2.MALE.getDescription());
}
```

![运行结果](/enum/enum-test-2.png)

&emsp;&emsp;首先在枚举类中定义一个变量,安全起见,这个变量最好是私有的,通过构造器传递形参,然后将这个形参赋值给我们定义的私有属性,在我们的枚举变量后面使用括号来定义中文描述,再实现这个私有属性的getter方法以便外界获取.这样,我们就可以获取到我们枚举变量的中文描述啦.

### 重写父类方法

&emsp;&emsp;所有的枚举类都是继承自`Enum`类的,而`Enum`类中只有`toString()`方法不是用final修饰的,也就是说,我们自定义枚举类,只能重写`toString()`方法.

- 重写`toString()`方法

```java
enum Sex2{
    MALE("男"),
    FEMALE("女");

    private String description;

    private Sex2(String description){
        this.description = description;
    }

    public String getDescription(){
        return description;
    }

    @Override
    public String toString() {
        return "Sex2{" +
                "description='" + description + '\'' +
                '}';
    }
}

@Test
public void sfsnkjc(){
    System.out.println("Sex2.FEMALE -> "+Sex2.FEMALE);
    System.out.println("Sex2.FEMALE.name() -> "+Sex2.FEMALE.name());
    System.out.println("Sex2.FEMALE.toString() -> "+Sex2.FEMALE.toString());
    System.out.println("Sex2.FEMALE.ordinal() -> "+Sex2.FEMALE.ordinal());
    System.out.println("Sex2.valueOf(\"FEMALE\") -> "+Sex2.valueOf("FEMALE"));
    System.out.println("Enum.valueOf(Sex2.class,\"FEMALE\") -> "+Enum.valueOf(Sex2.class,"FEMALE"));
    System.out.println("Arrays.toString(Sex2.values()) -> "+Arrays.toString(Sex2.values()));

    System.out.println(Sex2.MALE.getDescription());
}
```

![运行结果](/enum/enum-test-3.png)

&emsp;&emsp;根据结果我们知道,直接电泳枚举类名点枚举变量的方式实际上就是调用`toString()`方法,`valueOf()`方法也是调用的`toString()`方法.

### 枚举中的抽象方法

&emsp;&emsp;在枚举中,有时候我们每一个枚举变量可能需要进行不同的操作,这时,我们可以在枚举类中定义一个抽象方法,然后每一个枚举变量都实现此抽象方法,在实现抽象方法的时候进行不同的以满足我们的需求.

- 定义抽象方法

```java
public enum Sex {
    MALE("男"){
        @Override
        public String info() {
            return "This is male";
        }
    },
    FEMALE("女"){
        @Override
        public String info() {
            return "This is female";
        }
    };

    private String description;

    private Sex(String description){
        this.description = description;
    }

    public String getDescription(){
        return description;
    }

    public abstract String info();

    @Override
    public String toString() {
        return "Sex{description='" + description + "'}";
    }
}
```

- 使用

```java
@Test
public void sbkcj(){
    Sex male = Sex.MALE;
    Sex female = Sex.FEMALE;
    System.out.println(male.info());
    System.out.println(female.info());
}
```

![运行结果](/enum/enum-abstract.png)

&emsp;&emsp;需要注意的是,此抽象方法的访问限制符大多数情况还是需要给public的,如果你省略访问修饰符不写那么此方法只能包下可见,包外的类中就访问不到这个方法了.


&emsp;&emsp;既然枚举变量可以重写枚举类中的抽象方法,那么其他的方法呢?

- 枚举变量重写枚举类的方法

```java
public enum Sex {
    MALE("男"){
        @Override
        public String info() {
            return "This is male";
        }

        @Override
        public String getDescription() {
            return super.getDescription();
        }

        @Override
        public String toString() {
            return "haha";
        }
    },
    FEMALE("女"){
        @Override
        public String info() {
            return "This is female";
        }
    };

    private String description;

    private Sex(String description){
        this.description = description;
    }

    public String getDescription(){
        return description;
    }

    public abstract String info();

    @Override
    public String toString() {
        return "Sex{description='" + description + "'}";
    }
}
```

&emsp;&emsp;具体调用和上例中调用抽象方法的方式是一样的,这里就不展示运行结果了.

&emsp;&emsp;到这里,给我的感觉就是枚举变量很像是这个枚举类的一个子类,枚举变量可以实现枚举类的抽象方法,可以重写枚举类的方法,而且用IDE重写默认实现是调用`super.方法签名`的方式,像极了子类重写父类方法的形式,那么问题来了,可以在枚举变量中定义属性么?

- 枚举变量中定义属性及方法

```java
public enum Sex {
    MALE("男"){
        private String characteristic;

        private long number = 1;

//        public static String HOBBY = "female";

        {
            number++;
        }
        
        /*static{
            
        }*/

        public String getCharacteristic() {
            return characteristic;
        }

        public void setCharacteristic(String characteristic) {
            this.characteristic = characteristic;
        }

        public long getNumber(){
            return number;
        }

        @Override
        public String info() {
            return "This is male";
        }

        @Override
        public String getDescription() {
            return super.getDescription();
        }

        @Override
        public String toString() {
            return "haha";
        }
    },
    FEMALE("女"){
        @Override
        public String info() {
            return "This is female";
        }
    };

    private String description;

    private Sex(String description){
        this.description = description;
    }

    public String getDescription(){
        return description;
    }

    public abstract String info();

    @Override
    public String toString() {
        return "Sex{description='" + description + "'}";
    }
}
```

&emsp;&emsp;惊奇的发现,真的可以定义属性和方法以及初始化域,但是!静态的属性,方法,域是不可以的,会报编译错误.这样看来,反而觉得上面的关于枚举变量类似枚举类的子类的臆想不准确,其实枚举变量更像是枚举类的一个内部类.

![运行结果](/enum/enum-inner.png)

&emsp&emsp;在枚举变量中定义私有属性,初始化域,方法没有报错,编译也能正常的通过,但是这里我遇到了一个问题,除了实现的抽象方法以及重写的枚举类的方法外我并不能获取我定义的方法.

![运行结果](/enum/enum-can-not-get.png)

&emsp;&emsp;既然常规方法不能,那么这里我换一种方式,通过反射获取方法然后调用.

- 反射获取方法

```java
@Test
public void ancsjaknc(){
    Sex male = Sex.MALE;
    Class clazz = male.getClass();
    try {
        Method method = clazz.getMethod("getNumber");
        System.out.println(method);
        System.out.println(method.invoke(male));
    } catch (NoSuchMethodException e) {
        System.out.println("未找到该方法");
    } catch (IllegalAccessException e) {
        System.out.println("方法调用出错");
    } catch (InvocationTargetException e) {
        System.out.println("方法调用出错");
    }
}
```

![运行结果](/enum/enum-reflect.png)

&emsp;&emsp;想法是美好的,现实是残酷的.确实,我们可以通过反射获取该方法,但是在使用`invoke()`方法调用方法时却出问题了.对于这样的情况我心里有个大概的猜想,但是为了不误导,此处就不多提及.到此,我们知道枚举变量中可以重写枚举类的方法,不管是抽象的还是非抽象的都可以实现,调用也是没有问题的,至于定义私有属性和公有方法暂且当不可能吧,毕竟虽然可以编译通过,但是并不能调用.

### 枚举和接口

&emsp;&emsp;之前有提到过,所有的枚举类都是继承自`Enum`类,而java都是单继承的,所以枚举类不能继承,但是可以实现多接口.

- 实现接口

```java
interface SexDescription{
    String info();
}
enum Sex3 implements SexDescription{
    MALE,FEMALE;

    @Override
    public String info() {
        if(StringUtils.equals(this.name(),MALE.name())){
            return "男";
        }else if(StringUtils.equals(this.name(),FEMALE.name())){
            return "女";
        } else {
            return "未知";
        }
    }
}

@Test
public void dsbjj(){
    Sex3 sex3 = Sex3.MALE;
    System.out.println(sex3.info());
}
```

![运行结果](/enum/enum-interface-1.png)



