---
title: String
date: 2018-12-17 18:47:47
tags:
    - java字符串创建
    - java字符串常用方法
categories: java
---

&emsp;&emsp;在这里记录一下自己对java字符串的学习过程，记录的同时也加深印象。

<!-- more -->

### 字符串创建

&emsp;&emsp;我们平时使用String str = "This is a string."这种方式创建的字符串，JVM会将该字符串存储在常量池中，而使用String str = new String("This is a string.")创建的字符串和普通的对象一样，会在堆空间开辟一块空间存储该对象，然后在栈空间开辟一块空间存储该对象的地址的值。

- 例子

```java
public class StringTest {
    @Test
    public void test(){
        String str = "This is a string.";
        String str1 = "This is a string.";
        String str2 = "This is " + "a string.";
        String str3 = new String("This is a string.");
        System.out.println(str == str1);
        System.out.println(str == str2);
        System.out.println(str == str3);
    }
}
```

![测试结果](/String/String-create-test.png)

&emsp;&emsp;JVM在创建这些对象的时候首先在常量池中查找这些字面量是否存在，如果存在，那么该对象直接引用这个字面量即可，如果不存在那么在常量池中创建这个字面量，然后该对象引用这个字面量。

&emsp;&emsp;本例中，在创建str对象时，JVM在常量池中没有找到"This is a string"这个字面量，那么先在常量池创建这个字面量，然后变量名为str的对象引用这个字面量。创建变量名为str2的对象时在常量池中找到了这个字面量，所以直接引用这个字面量即可，所以，str和str2的地址值一样。在创建变量名为str3的对象时，首先在常量池寻找"This is "这个字面量，没有找到，那么创建该字面量，再寻找"a string."这个字面量，没有找到，再创建该字面量，然后将这两个字面量进行拼接之后形成新的字面量"This is a string."，在常量池中寻找到该字面量，那么直接引用这个字面量，所以str2引用的地址值和str以及str1一样，但是在创建str2这个对象的时候又在常量池中创建了两个字面量，这就是str2和str以及str1创建过程的不同之处。在创建str3对象时，使用了`public String(String original)`这个构造器来创建对象，很显然，这个构造器就是将一个字符串赋值给另一个对象，但是地址值发生变化，毕竟new了一个对象。

### 字符串常用方法

#### 字符串分割方法

- 返回值类型+方法签名

```java
String[] split(String regex)
String[] split(String regex, int limit)
```

- String regex 正则表达式分隔符

- int limit 分割的份数

##### 用法

&emsp;&emsp;在java中，使用`split()`方法可以将字符串按照指定的分割符进行分割，String类中重载了该方法。可以看一下一个参数的方法实现。

```java
public String[] split(String regex) {
    return split(regex, 0);
}
```

&emsp;&emsp;很显然，传递一个参数调用了传递两个参数的方法，int型的值传递0，也就是说使用regex分割符将字符串分割，返回分割后的不限制分割份数的字符串数组。

- 看一下传递一个参数的方法，即不考虑分隔份数的情况下方法的使用。

```java
@Test
public void test(){
    String str = "I-love-you";
    String[] strs = str.split("-");
    System.out.println("/*---------------------分割符---------------------------*/");
    System.out.println("使用普通的字符进行字符串的分隔\t长度："+strs.length);
    for(String s : strs){
        System.out.println(s);
    }

    String str1 = "ferry.org.cn";
    String[] strs1 = str1.split("\\.");
    System.out.println("/*---------------------分割符---------------------------*/");
    System.out.println("使用转义字符进行字符串的分隔\t长度："+strs1.length);
    for(String s : strs1){
        System.out.println(s);
    }

    String sql = "1 = 1 and 2 = 2 or 3 = 3 or4 = 4";
    String[] sqls = sql.split("and |or ");
    System.out.println("/*---------------------分割符---------------------------*/");
    System.out.println("使用多个字符进行字符串的分隔\t长度："+sqls.length);
    for(String s : sqls){
        System.out.println(s);
    }
}
```

![运行结果](/String/String-split-test.png)

&emsp;&emsp;如果字符串是使用转义字符进行连接的，我们需要以这些转义字符进行字符串的分割，那么需要在转义字符的前面加上`\\`，不然该分割符是无效的。

&emsp;&emsp;上面最后一个例子需要注意一下，split方法中可以使用`|`字符将多个字符作为分隔符。而本例中我使用了两个分割符对该字符串进行分割，`and `和`or `，注意，and和or后面都是有空格的，所以，本例中最后一个or4 = 4并没有被分割，因为or和4中间并没有空格。

&emsp;&emsp;值得一提的是，如果你需要进行分割的字符串并不包含你给定的分割符，那么它会返回一个长度为1的字符串数组，该数组中的唯一一个元素就是待分割的字符串。

```java
@Test
public void test(){
    String str = "I love you";
    String[] strs = str.split("-");
    System.out.println("/*---------------------分割符---------------------------*/");
    System.out.println("使用普通的字符进行字符串的分隔\t长度："+strs.length);
    for(String s : strs){
        System.out.println(s);
    }
}
```

![运行结果](/String/String-split-test1.png)

&emsp;&emsp;还有一种可能是需要注意一下的，当你使用多个分割符时，如果待处理的字符串中这几种分割符正好在一起，那么会发生什么呢，看一个例子。

```java
@Test
public void test(){
    String str = "ferry.org.cn";
    String[] strs = str.split("\\.|org");
    System.out.println("/*---------------------分割符---------------------------*/");
    System.out.println("长度："+strs.length);
    for(String s : strs){
        System.out.println(s);
    }
}
```

![运行结果](/String/String-split-test3.png)

&emsp;&emsp;运行结果就是：分割成了四个字符串，其中两个为空字符串。那么这个结果怎么产生的呢？这是因为在分割这个字符串时遇到第一个分割符`"."`，得到第一个字符串"ferry"和剩余字符串"org.cn"，接着执行遇到第二个分割符`"org"`,这个分割符位于字符串起始位置，那么就会分割出一个空字符串然后得到剩余字符串".cn"，继续执行，遇到第三个分割符`"."`，位于字符串起始位置，分割得到空字符串和剩余字符串，继续执行，未找到分割符，将剩余字符串添加到集合中并转换成数组返回。

- 指定分割份数的分割方法

&emsp;&emsp;指定分割份数我们考虑四种情况，第一种：limit为负值；第二种：limit为0；第三种：limit值在待分割的字符串能分割的份数的有效范围内；第四种：limit值大于待分割的字符串能分割的份数。当然，第二种情况不用考虑了，前面只传递一个参数分割字符串的正则表达式调用的就是limit=0的方法，该种情况下不考虑分割份数，按给定的正则表达式分割并返回分割后的字符串数组。

```java
@Test
public void test(){
    String str1 = "I-love-you";
    String[] strs1 = str1.split("-", -1);
    System.out.println("/*---------------------分割符---------------------------*/");
    System.out.println("分割份数为负值：\t长度："+strs1.length);
    for(String s : strs1){
        System.out.println(s);
    }

    String str2 = "I-love-you";
    String[] strs2 = str2.split("-", 2);
    System.out.println("/*---------------------分割符---------------------------*/");
    System.out.println("分割份数在待分割字符串能分割的份数的范围内：\t长度："+strs2.length);
    for(String s : strs2){
        System.out.println(s);
    }

    String str3 = "I-love-you";
    String[] strs3 = str3.split("-", 999);
    System.out.println("/*---------------------分割符---------------------------*/");
    System.out.println("分割份数大于待分割字符串能分割的份数：\t长度："+strs3.length);
    for(String s : strs3){
        System.out.println(s);
    }
}
```

![运行结果](/String/String-split-test2.png)

- 当limit为负值时，和limit=0返回结果一样，都是能分割几份就分割几份。

- 当limit在分割份数有效范围内时，前面几份分割按指定分割符分割，最后一份则为剩余未分割的字符串。

- 当limit大于有效分割份数时，分割结果与limit=0结果一样。

#### 字符串比较是否相等

- 返回值类型+方法签名

```java
boolean equals(Object anObject);
```

- Object anObject: 待比较的Object对象

##### 用法

- 使用案例

```java
@Test
public void test(){
    String str = "ferry";
    Object obj = "ferry";
    System.out.println(str.equals(obj));
}
```

![运行结果](/String/string-equals.png)

##### 源码分析

- 源码

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

&emsp;&emsp;自上而下：

1. 判断两个对象地址值是否相等，相等返回true，否则判断2；

2. 判断形参类型是否为字符串，不是字符串返回false，是判断3；

3. 将形参强制类型转化为字符串，判断两个字符串长度是否相等，不相等返回false，相等判断4；

4. 循环比较两个字符串中的每一个字符是否相等，遇到不相等的，方法返回false，全部相等返回true。

#### 字符串忽略大小写比较是否相等

- 返回值类型+方法签名

```java
boolean equalsIgnoreCase(String anotherString);
```

- String anatherString: 待比较的字符串对象

##### 用法

- 使用案例

```java
@Test
public void test(){
    String str = "FERRY";
    String msg = "ferry";
    System.out.println(str.equalsIgnoreCase(msg));
}
```

![运行结果](/String/string-equalsIgnore.png)

##### 源码分析

- 源码

```java
public boolean equalsIgnoreCase(String anotherString) {
    return (this == anotherString) ? true
            : (anotherString != null)
            && (anotherString.value.length == value.length)
            && regionMatches(true, 0, anotherString, 0, value.length);
}
```

&emsp;&emsp;这个方法的实现很简单，分为几个步骤：

1. 判断两个字符串的引用是否相等，相等返回true；

2. 判断形参是否为空，若为空返回false，否则判断3；

3. 判断两个字符串长度是否相等，相等判断4，否则返回false；

4. 调用`regionMatches()`方法判断每个字符是否相等。

&emsp;&emsp;`regionMatches(boolean ignoreCase, int toffset, String other, int ooffset, int len)`方法的作用是判断一个区域内两个字符串是否相等，相等返回true，否则返回false。当形参ignoreCase为true时忽略大小写进行比较。

#### 字符串比较某个区域内是否相等

- 返回值类型+方法签名

```java
boolean regionMatches(int toffset, String other, int ooffset, int len);
boolean regionMatches(boolean ignoreCase, int toffset, String other, int ooffset, int len);
```

- boolean ignoreCase: 是否忽略大小写

- int toffset: 调用方法的字符串的子区域起始偏移量

- String other: 待比较字符串

- int ooffset: 待比较字符串子区域起始偏移量

- int len: 要比较的字符数量

##### 用法

- 使用案例

```java
@Test
public void test(){
    String str = "I love ferry";
    String msg = "You love ferry";
    System.out.println(str.regionMatches(2, msg, 4, 4));
}
```

![运行结果](/String/String-rem-1.png)

&emsp;&emsp;偏移量的理解，本例中调用该方法的字符串偏移量为2，也就是偏移两个字符，从第三个字符开始比较，同样，msg字符偏移量为4，就是偏移4个字符，从第五个字符开始比较。其实也就是比较`"love"`和`"love"`是否相等。

- 忽略大小写比较两个字符串某一区域是否相等

```java
@Test
public void test() {
    String str = "I love ferry";
    String msg = "You LOVE ferry";
    System.out.println(str.regionMatches(true, 2, msg, 4, 4));
}
```

![运行结果](/String/String-rem-2.png)

##### 源码分析

- 源码

```java
public boolean regionMatches(int toffset, String other, int ooffset, int len) {
    char ta[] = value;
    int to = toffset;
    char pa[] = other.value;
    int po = ooffset;
    if ((ooffset < 0) || (toffset < 0)
            || (toffset > (long)value.length - len)
            || (ooffset > (long)other.value.length - len)) {
        return false;
    }
    while (len-- > 0) {
        if (ta[to++] != pa[po++]) {
            return false;
        }
    }
    return true;
}
```

&emsp;&emsp;自上而下：

1. 获取两个字符串的字符数组和对应的比较字符的起始位置，判断2；

2. 如果两个起始位置不合法，返回false，否则判断3；

3. 循环判断两个字符串对应的字符是否相等，不相等返回false，循环结束返回true，循环次数为比较长度。

- 源码

```java
public boolean regionMatches(boolean ignoreCase, int toffset, String other, int ooffset, int len) {
    char ta[] = value;
    int to = toffset;
    char pa[] = other.value;
    int po = ooffset;
    if ((ooffset < 0) || (toffset < 0)
            || (toffset > (long)value.length - len)
            || (ooffset > (long)other.value.length - len)) {
        return false;
    }
    while (len-- > 0) {
        char c1 = ta[to++];
        char c2 = pa[po++];
        if (c1 == c2) {
            continue;
        }
        if (ignoreCase) {
            char u1 = Character.toUpperCase(c1);
            char u2 = Character.toUpperCase(c2);
            if (u1 == u2) {
                continue;
            }
            if (Character.toLowerCase(u1) == Character.toLowerCase(u2)) {
                continue;
            }
        }
        return false;
    }
    return true;
}
```

&emsp;&emsp;自上而下：

1. 获取两个字符串的字符数组和对应的比较字符的起始位置，判断2；

2. 如果两个起始位置不合法，返回false，否则判断3；

3. 循环判断每个字符是否相等，循环次数为比较长度，循环结束返回true：

    - 循环中获取待比较的两个字符；
    
    - 判断两个字符是否相等，相等结束本次循环开始下次循环，不相等进行下一步判断；
    
    - 判断是否忽略大小写进行比较，否返回false，是将两个字符都转化成大写进行比较，相等结束本次循环，开始下一此循环，否将两个字符转化为小写进行比较，相等结束本次循环进行下一次循环，否返回false。


#### 字符串截取

- 返回值类型+方法签名

```java
String substring(int beginIndex);
String substring(int beginIndex, int endIndex)
```

- int beginIndex: 截取起始位置

- int endIndex: 截取结束为止

##### 用法

- 使用案例

```java
@Test
public void test() {
    String str = "I love ferry";
    System.out.println(str.substring(2));
    System.out.println(str.substring(2,6));
}
```

![运行结果](/String/String-substring.png)

&emsp;&emsp;截取字符串的起始位置和结束为止的包含关系：[beginIndex,endIndex)。如果起始位置数值上等于字符串长度，那么截取的就是一个空字符串。

##### 源码分析

- 源码

```java
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    int subLen = value.length - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
```

&emsp;&emsp;自上而下：

1. 如果起始位置小于0，抛出下标越界异常，否则判断2；

2. 如果截取的字符串长度小于0，抛出下标越界异常，否则判断3；

3. 如果起始位置为0，返回调用方法的字符串，否则调用构造器`new String(char value[], int offset, int count)`生成字符串。

- 源码

```java
public String substring(int beginIndex, int endIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    if (endIndex > value.length) {
        throw new StringIndexOutOfBoundsException(endIndex);
    }
    int subLen = endIndex - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return ((beginIndex == 0) && (endIndex == value.length)) ? this
            : new String(value, beginIndex, subLen);
}
```

&emsp;&emsp;自上而下：

1. 如果起始位置小于0或者结束位置大于字符串长度，抛出下标越界异常，否则判断2；

2. 如果截取的字符串长度小于0，抛出下标越界异常，否则判断3；

3. 如果起始位置为0，结束位置数值上等于字符串长度，则返回调用方法的字符串，否则调用构造器`new String(char value[], int offset, int count)`生成字符串。



#### 字符(字符串)替换

- 返回值类型+方法签名

```java
String replace(char oldChar, char newChar);
String replace(CharSequence target, CharSequence replacement);
```

- char oldChar: 待替换的字符

- char newChar: 替换字符

- CharSequence target: 待替换的字符序列

- CharSequence replacement: 替换字符序列

##### 用法

- 使用案例

```java
@Test
public void replaceTest(){
    String str = "I love you";
    System.out.println(str.replace(' ', '-'));
    System.out.println(str.replace("love", "miss"));
}
```

![运行结果](/String/String-replace-char.png)

&emsp;&emsp;字符串替换方法中,可以传递字符,字符串,该方法将所有值为oldChar/target的字符/字符串替换为newChar/replacement.

##### 源码分析

- 源码

```java
public String replace(char oldChar, char newChar) {
    if (oldChar != newChar) {
        int len = value.length;
        int i = -1;
        char[] val = value; /* avoid getfield opcode */

        while (++i < len) {
            if (val[i] == oldChar) {
                break;
            }
        }
        if (i < len) {
            char buf[] = new char[len];
            for (int j = 0; j < i; j++) {
                buf[j] = val[j];
            }
            while (i < len) {
                char c = val[i];
                buf[i] = (c == oldChar) ? newChar : c;
                i++;
            }
            return new String(buf, true);
        }
    }
    return this;
}
```

&emsp;&emsp;自上而下：

1. 如果待替换字符和替换字符不相等返回调用方法的字符串,否则判断2;

2. 对字符数组进行循环,如果待替换字符与字符串的字符数组中任意一个都不相等则返回调用方法的字符串,否则得到第一个相等的字符下标,判断3;

3. 循环将第一个相等的字符前面的字符复制到新的字符数组中,判断4;

4. 从第一个相等的字符位置开始循环将后面的字符数组复制到新的字符数组中若待复制字符等于oldChar则替换成newChar,使用构造器返回替换完成的字符串.

&emsp;&emsp;实际上整个流程就是如果字符串中没有oldChar,那么返回本字符串,如果有,就先复制第一个字符等于oldChar之前的字符数组,然后剩下的字符数组在循环中替换字符,然后通过构造器将字符数组转换成字符串返回.

- 源码

```java
public String replace(CharSequence target, CharSequence replacement) {
    return Pattern.compile(target.toString(), Pattern.LITERAL).matcher(
            this).replaceAll(Matcher.quoteReplacement(replacement.toString()));
```

&emsp;&emsp;权衡利弊之后,不能解析了...时间紧张...


#### 字符串移除前后空格

- 返回值类型+方法签名

```java
String trim();
```

##### 用法

- 使用案例

```java
@Test
public void trimTest(){
    String str = " I love ferry ";
    System.out.println("before length:"+str.length());
    System.out.println(str.trim());
    System.out.println("after length:"+str.trim().length());
}
```

![运行结果](/String/String-trim.png)

&emsp;&emsp;`trim()`方法可以将字符串的前导空格、尾随空格和行终止符移除。

##### 源码分析

- 源码

```java
public String trim() {
    int len = value.length;
    int st = 0;
    char[] val = value;    /* avoid getfield opcode */

    while ((st < len) && (val[st] <= ' ')) {
        st++;
    }
    while ((st < len) && (val[len - 1] <= ' ')) {
        len--;
    }
    return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
}
```

&emsp;&emsp;自上而下：

1. 循环判断字符串的前面的字符是不是空格,是`st`加一;

2. 循环判断字符串的后面的字符是不是空格,是`len`减一;

3. 如果字符串没有前导空格或尾随空格或者行终止符则返回调用方法的字符串,否则返回截取后的字符串;


