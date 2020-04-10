---
title: java集合
date: 2018-12-17 18:36:29
tags:
    - ArrayList
    - HashMap
categories: java
---

&emsp;&emsp;java集合的学习记录...

<!-- more -->


### ArrayList

&emsp;&emsp;ArrayList集合是List接口的可变长数组的实现。他的底层是用数组实现的。

#### 特点

- 集合中允许null值存在；

- 集合容量可变，自增长，约1.5倍；

- 非线程安全。

#### 初始化

- 属性列表

```java
private static final int DEFAULT_CAPACITY = 10;

private static final Object[] EMPTY_ELEMENTDATA = {};

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

transient Object[] elementData;

private int size;
```

&emsp;&emsp;这里面有一个很重要的属性`elementData`,该集合对象的所有的值都是存储在这个数组对象中的,只是将存值和取值的实现进行了一定的封装.可以看到,这个属性是反序列化的,这说明ArrayList这个类一定实现了`java.io.Serializable`接口.

- 构造器源码

```java


public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

&emsp;&emsp;当使用空构造器时，会将类的静态常量所引用的空数组的地址值赋给`elementData`属性,此时,该对象被初始化为一个空集合.当使用传递一个int型参数的构造器时,若传参不合法抛出异常,传参为0初始化为一个空集合,传参大于0,则将`elementData`数组初始化,长度为传递的参数大小.第三种形式就是传递一个集合,将这个集合初始化.

#### 数据存储

&emsp;&emsp;ArrayList添加元素有`set`和`add`方法

















































