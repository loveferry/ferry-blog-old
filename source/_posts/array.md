---
title: 数组
date: 2018-12-17 18:35:29
categories: java
tags:
    - java数组
    - Arrays类
---

&emsp;&emsp;数组是相同类型的、用一个标识符名称封装到一起的一个对象序列或基本类型数据序列。

<!-- more -->

### 数组对象的定义

&emsp;&emsp;在java中,数组有两种声明方式

- 声明数组

```java
int[] a; // java推荐
int b[]; // C语言形式,不推荐
```

&emsp;&emsp;在java中,推荐使用第一种形式声明一个数组的引用.使用数据类型+中括号([])+空格+变量名来声明一种类型的数组形式.此时,java虚拟机会在栈空间开辟一块空间来存放我们声明的变量,也就是我们的引用的地址值.接下来,就需要初始化一个数组并使用我们声明的数组变量引用这个数组.

- 初始化数组并引用

```java
a = new int[10];
```

&emsp;&emsp;在java中,按照优先级,会先执行赋值符号右边的表达式,`new int[10]`是初始化一个长度为10的整型数组,这步操作将在堆空间开辟一块相应大小的内存,其中,数组的每一个元素都赋予默认值,int型的默认值为0,其他类型的默认值可以自行创建对应的数组类型并初始化然后debug查看.第二步执行赋值操作,将初始化的数组的第一个元素的地址的值赋值给栈空间的数组变量,所以,栈空间中存放的是堆空间开辟的内存的对应元素的地址的值.

&emsp;&emsp;初始化有三种方式,第一种方式如上,创建一个具有默认值的数组,使用这种方式要求必须在中括号中指定数组的大小.接下来介绍另外两种.

- 数组初始化方式二

```java
b = new int[]{1,2,3,4,5};
```

&emsp;&emsp;这种方式在初始化的时候就赋值,数组的长度即为初始化时大括号中元素的个数.注意!指定数组长度和赋默认值不能同时进行,也就是说使用方式二不能指定数组长度.

- 数组初始化方式三

```java
int[] c = {1,2,3};
```

&emsp;&emsp;这种方式是第二种方式的一种简写形式,数组的长度为初始化元素的个数.但是这种形式有一个问题,不能先声明再初始化,必须用一个表达式完成声明和初始化的工作.

- 以上方式声明并初始化数组的debug效果

![运行结果](/array/array-init.png)

&emsp;&emsp;当然,我们平时定义数组为了安全考虑声明和初始化是同时进行的,例如`int[] a = new int[10];`,防止在声明数组变量但未初始化的时候就使用该变量从而引发空指针异常.

&emsp;&emsp;这里说几点:

1. `int[] a = new int[0];`和`int[] b = new int[]{};`和`int[] c = {};`都是声明一个长度为0的数组,也称之为空数组.注意:空数组不意味着它是null,空数组有点类似字符串的空字符串,在实际应用中,在方法中需要返回数组元素,但是经过操作并不能得到想要的数组元素,但是总不能返回null值吧,所以我们可以返回一个空数组.

2. `new int[]{1,2,3};`是一个匿名数组,因为没有任何一个变量指向这个数组,它的性质和匿名对象一样,离开当前代码行,就没办法再使用这个匿名数组了,因为再也找不到它了.

3. 数组是一个类,实例化一个数组就是创建了一个数组的对象!!!

### 数组使用

&emsp;&emsp;通过`变量名[下标]`获取数组中对应位置的元素,下标从0开始.

&emsp;&emsp;使用数组时可以通过`a[4] = 10;`形式实现对数组中的元素的操作,当然,这样做是有很大的风险的,当下标不合法时会抛出数组下标越界异常.

- 数组复制

```java
@Test
public void fdshfiue(){
    int[] a = {43,4454,23,1225,3};
    int[] b = a;
    System.out.println(Arrays.toString(a));
    System.out.println(Arrays.toString(b));
    b[1] = 1;
    System.out.println(Arrays.toString(a));
    System.out.println(Arrays.toString(b));
}
```

![迎新结果](/array/array-copy-error.png)

&emsp;&emsp;前面说过,数组是一个对象,所以将一个数组使用赋值符号直接赋值给另一个变量名,那么实际上也就是将该数组对象的地址值赋值给该变量,所以这两个变量引用的是同一个数组对象,无论哪个变量对数组对象中的元素进行了修改,本质上都是对同一个对象进行修改.所以如果我们想要对数组进行复制并再复制体上进行操作可以使用下面要说的复制方法.

#### 数组长度

- 获取数组的长度

```java
@Test
public void bcsjh(){
    int[] a = new int[]{1,2,3,4,5};
    int length = a.length;
    System.out.println(length);
}
```

&emsp;&emsp;`length`是数组类的成员变量,所以可以直接使用变量名.length的方式获取数组的长度.

#### 数组的循环

&emsp;&emsp;当我们需要对数组中的每一个元素都进行统一操作的时候,就需要使用循环来完成了.

&emsp;&emsp;数组的循环有两种形式,一种是使用普通的循环方式,另一种是使用增强for循环.

```java
@Test
public void bcsjh(){
    int[] a = new int[]{1,2,3,4,5};
    int length = a.length;
    System.out.println("数组的长度: " + length);
    // 一般形式的循环
    for (int i = 0; i < length; i++) {
        System.out.println("数组中第 " + i + " 个元素: " + a[i]);
    }
    // 增强for循环
    for (int value: a){
        ++value;
        System.out.println(value);
    }
}
```

![运行结果](/array/array-for.png)

&emsp;&emsp;一般形式的循环有三种,`do{...}while(boolean)`, `while(boolean){...}` 和 `for(初始化;判断;操作){...}`,当我们需要在循环中使用数组的下标时那么我们用for循环肯定更加方便一些,而不需要使用下标则使用增强for循环更加简洁.另外两种循环在数组应用上稍显复杂,这里不推荐.

#### 数组的输出

- 数组输出的几种形式

```java
@Test
public void dbscjdsn(){
    int[] a = new int[]{1,2,3,4,5};
    System.out.println(a.toString());
    for (int i : a) {
        System.out.println(i);
    }
    System.out.println(Arrays.toString(a));
}
```

![运行结果](/array/array-toString.png)

&emsp;&emsp;数组是一个类,而数组类并没有重写Object类的toString()方法,所以会直接输出类的信息,而数组类是JVM创建的所以类名才会如此奇怪.我们也可以选择使用循环进行每一个数组元素的输出,或者使用Arrays类的toString()方法进行数组的输出.

#### 数组的比较

- 数组比较的几种方式

```java
@Data
class User{
    private String name;

    public User(String name){
        this.name = name;
    }
}

@Test
public void fsjbdjvcb(){
    User[] users = {new User("ferry"),new User("honey")};
    User ferry = new User("ferry");
    User honey = new User("honey");
    User[] us = new User[]{ferry,honey};
    System.out.println(users == us);
    System.out.println(us.equals(users));
    System.out.println(Arrays.equals(us, users));
}
```

![运行结果](/array/array-equals.png)

&emsp;&emsp;使用`==`进行比较就是比较两个对象的地址的值,显而易见,两个不同的对象地址值自然不相同;使用`对象.equals(对象)`的方式进行比较,由于枚举类没有重写Object对象的equals()方法,所以直接调用的Object类的equals()方法,实际上也就是使用`==`比较两者的地址的值.使用Arrays类的equals方法进行比较将进行数组对象中每一个元素的值进行比较,所以这种比较方式才是我们正常发开时所需要的方式.

&emsp;&emsp;到这里,已经使用了Arrays类的两个方法,toString()和equals()方法都非常的实用.那么,Arrays类中还有什么方法可以让我们更加高效的处理数组呢?

### Arrays类

&emsp;&emsp;Arrays类是数组的操作类，定义在java.util包中，主要的功能可以实现数组元素的查找，数组内容的填充、排序等。该类中的绝大多数方法都是静态的.通过`Arrays.方法名`形式调用.

#### `toString()`方法

- 返回数组的字符串描述,返回字符串:此方法被重载多次,支持多种数据类型

```java
System.out.println(Arrays.toString(new long[]{1L,10L}));
// 运行结果: [1, 10]
```

#### `equals()`方法

- 比较两个数组对象是否相等,返回布尔值:此方法被重载多次,支持多种数据类型

```java
System.out.println(Arrays.equals(new String[]{"ferry","honey"},new String[]{"ferry","honey"}));
// 运行结果: true
```

#### `sort()`方法和`parallelSort()`方法

- 数组排序,无返回值: 此方法被重载多次,支持多种数据类型

```java
@Data
class User implements Comparable<User>{
    private String name;

    public User(String name){
        this.name = name;
    }

    @Override
    public int compareTo(User user) {
        return StringUtils.compare(name,user.getName());
    }
}

@Test
public void cbsj(){
    User[] users = new User[]{new User("ferry"),new User("joyce"),new User("honey")};
    Arrays.sort(users);
    System.out.println(Arrays.toString(users));
}
```

![运行结果](/array/array-sort.png)

&emsp;&emsp;如果使用java类数组进行排序,那么要求这个类必须实现了`Comparable`接口实现其方法`compareTo()`,如果没有实现这个方法那么运行时会抛出异常.

&emsp;&emsp;sort()方法默认使用升序排序,即从小到大顺序,其中,可使用重载的方法`void sort(Object[] a, int fromIndex, int toIndex)`选择数组的一个区间进行排序.

&emsp;&emsp;java8以后如果处理的数据量过大,建议使用parallelSort()方法,该方法实现功能和sort方法一样,但是实现方式不同,处理起大数据parallelSort()方法更为高效

#### `fill()`方法

- 数组填充,无返回值: 此方法被重载多次,支持多种数据类型

```java
@Test
public void asdia(){
    User[] users  = new User[3];
    User user = new User("ferry");
    Arrays.fill(users,user);
    System.out.println(Arrays.toString(users));
}
// 运行结果: [ArrayTest.User(name=ferry), ArrayTest.User(name=ferry), ArrayTest.User(name=ferry)]
```

&emsp;&emsp;该方法将指定变量填充到数组的每个元素上.Arrays类重载了三参的方法,可以选择数组一个区间进行填充元素.

#### `copyOf()`方法

- 复制数组，截取或用默认值填充(如有必要),以使副本具有指定的长度,返回复制后的数组: 此方法被重载多次,支持多种数据类型

```java
@Test
public void sajhbc(){
    User[] users = {new User("ferry"),new User("joyce"),new User("honey")};
    User[] copyUsers = Arrays.copyOf(users,5);
    System.out.println(Arrays.toString(copyUsers));
}
// 运行结果: [ArrayTest.User(name=ferry), ArrayTest.User(name=joyce), ArrayTest.User(name=honey), null, null]
```

#### `copyOfRange()`方法

-  将指定数组的指定范围复制到一个新数组,返回复制后的数组: 此方法被重载多次,支持多种数据类型

```java
@Test
public void sajhbc(){
    User[] users = {new User("ferry"),new User("joyce"),new User("honey")};
    User[] copyUsers = Arrays.copyOfRange(users,1,5);
    System.out.println(Arrays.toString(copyUsers));
}
// 运行结果: [ArrayTest.User(name=joyce), ArrayTest.User(name=honey), null, null]
```

&emsp;&emsp;`<T> T[] copyOfRange(T[] original, int from, int to)`,如果from小于0会抛出数组下标越界异常,如果to大于数组长度,则使用默认值进行新数组的填充,需要注意,to是不包含的.

#### `binarySearch()`方法

- 使用二分搜索法来搜索指定的数组,如果key在数组中，则返回搜索值的索引；否则返回-1或“-”（插入点）: 此方法被重载多次,支持多种数据类型

```java
@Test
public void cskbc(){
    int[] is = {2,6,2,6,5,21,76,4};
    Arrays.sort(is);
    System.out.println(Arrays.toString(is));
    System.out.println(Arrays.binarySearch(is, -4));
    System.out.println(Arrays.binarySearch(is, 4));
    System.out.println(Arrays.binarySearch(is, 8));
    System.out.println(Arrays.binarySearch(is, 100));
}
```

![运行结果](/array/array-binarySearch.png)

&emsp;&emsp;使用`binarySearch()`方法的数组应该是已经排好序的数组.此方法有重载形式可以在数组指定范围进行所搜.

> 当待搜索的值在数组中时返回搜索值出现的第一个的下标,
> 当待搜索的值不在数组中且值小于数组中最小的元素的值时返回-1,
> 当待搜索的值在不在数组中且值大于数组元素最小的值小于数组元素最大的值时返回**按顺序待插入的下标-1 的相反数**;
> 当待搜索的值在不在数组中且值大于数组元素最大的值时返回数组长度+1;

### 多维数组

&emsp;&emsp;多维数组是数组的数组。其实java只有一维数组，但是由于数组可以存放任意类型的数据，当然也就可以存放数组了，这个时候，就可以模拟多维数组了。

- 定义多维数组

```java
@Test
public void dsabcjh(){
    int[][] a = new int[4][4];
    int b[][] = new int[4][];
    int[][][] c = new int[5][9][5];
    int[][][] d = new int[5][9][];
    int e[][][] = new int[3][][];
}
```

&emsp;&emsp;多维数组的定义和一维数组的定义类似,我们推荐奖[]放在数据类型前面变量名后面的这种形式.在初始化的时候在指定后面纬度长度的前提是必须前面纬度的长度指定了,例如二维数组,不能指定二维长度而不指定一维长度.将数组举个例子.普通的数据类型就是一个点,一维数组就是一条线,二维数组就是一个以X轴和Y轴组成的面,三维数组就是以X轴,Y轴和Z轴组成的空间,至于更高维度,四维我们是不是可以想象成一个空间贯穿着一条时间轴构成空间随着时间的流逝产生的演变,五维就是一个空间比如地球随着时间流逝,发生在这片空间的历史...咳咳,扯远了...

- 多维数组方法调用

```java
@Test
public void dbajsbkc(){
    int[][] a = {{1,2,3,7},{3,6,3,1,7,9,3,43}};
    System.out.println(Arrays.toString(a));
}
```

![运行结果](/array/array-more-tostring.png)

&emsp;&emsp;显然,Arrays类对于数组的支持只是基于一维的,所以多维数组想要实现像一维数组那样的操作就需要使用循环了.

```java
@Test
public void dbajsbkcsd(){
    int[][] a = {{1,2,3,7},{3,6,3,1,7,9,3,43}};
    for (int[] ints : a) {
        System.out.println(Arrays.toString(ints));
    }
}
```

![运行结果](/array/array-more-tostring-2.png)

&emsp;&emsp;所以,Arrays类方法的支持想要对多维数组操作就需要使用循环降维.

### 数组类探索

&emsp;&emsp;数组是一个类,数组类是通过数据类型和纬度来确定具体是哪一个类的,不同的数据类型和纬度的数组所属的类是不一样的.

```java
@Test
public void sfjk(){
    int[] arr = {1,2,3};
    double[][] arr1 = new double[3][4];
    String[][][] arr2 = new String[7][5][6];
    System.out.println(arr.toString());
    System.out.println(arr1.toString());
    System.out.println(arr2.toString());
    System.out.println(arr.getClass().getSuperclass().getName());
}

// Object类toString实现
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

![运行结果](/array/array-class.png)

&emsp;&emsp;从结果可以推断出以下几点.

> 数组类是Object类的直接子类;
> 不同纬度的不同数据类型的数组对象属于不同的数组类;
> 数组类应该是有JVM生成的.

&emsp;&emsp;第一点好推断,此处不说,第二点:数组对象调用Object类的toString()方法拼接的是对象所属类的类名加上处理过后的哈希值,而上述例子使用了不同纬度的不同数据类型的数组对象打印出来的类名并不一致.再稍加分析推断,可以看到,'['的数量代表着数组的纬度,后面的'I,'D'等代表着数据类型.而这样的奇怪的非法的类名正常情况下是不存在的,能做到这一点的只有JVM了,所以得到第三点,数组类是由JVM在运行时生成的.

#### 协变的数组

&emsp;&emsp;数组的协变性是指：父类的数组引用可以指向子类的数组对象.

```java
@Data
class Base{
    private int id;
}
@Data
class Sub extends Base{
    private String name;
}
@Data
class Oth extends Base{
    private String hobby;
}

@Test
public void ssss(){
    Base[] arr = new Sub[5];
    arr[0] = new Oth();
    System.out.println(arr[0].toString());
}
```

&emsp;&emsp;Sub类Base类的子类,所以,Base类的数组引用可以指向Sub类的数组对象,但是,当你将引用中的某个元素指向该父类的其它子类时,编译器并不会报错,但是运行时会报错`java.lang.ArrayStoreException`,其实这一点就没有泛型好,泛型可以严格的检测类型是否一致,而且现在泛型的通配符的使用也实现了协变的功能.


```java
@Test
public void sssss(){
    Base[] arr = new Sub[5];
    System.out.println(arr.getClass());
    System.out.println(arr.getClass().getSuperclass());
}
```

![运行结果](/array/array-extend.png)

&emsp;&emsp;说明虽然引用是父类数组的引用,但是实际的类型还是子类对象的数组类.第二行输出更是说明所有的数组类的直接父类是Object类.

&emsp;&emsp;在本文的最后,友情提示一下,数组是不支持泛型的,究其原因,是因为数组在创建的时候就要知道它的数据类型,并且在对数组元素进行操作的时候都会检查类型,而泛型是用擦除实现的,在运行时泛型的类型会被擦除.你看,一个要求我必须知道元素类型,一个会把类型干掉,这不是打架了么,所以这两者是不能共存滴!