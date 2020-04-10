---
title: 正则表达式
date: 2020-03-30 11:20:11
categories: java
tags:
    - 正则表达式
    - Pattern类
    - Matcher类
---

&emsp;&emsp;字符串是编程中用到最多，最常见的一种数据结构，对字符串的处理需求无处不在。正则表达式是一种用来匹配字符串的强力武器，它的设计思想是用一种描述性语言给字符串定义一个规则，凡事符合规则的字符串，就认为"匹配"了。

<!-- more -->

# 正则表达式介绍

&emsp;&emsp;一个正则表达式通常被称为一个模式（pattern），为用来描述或者匹配一系列符合某个句法规则的字符串。例如：Handel、Händel和Haendel这三个字符串，都可以由H(a|ä|ae)ndel这个模式来描述。大部分正则表达式的形式都有如下的结构：
            
- 选择：竖线|代表选择（即或集），具有最低优先级。例如gray|grey可以匹配grey或gray。

- 数量限定：某个字符后的数量限定符用来限定前面这个字符允许出现的个数。最常见的数量限定符包括+、?和*（不加数量限定则代表出现一次且仅出现一次）：

> `+`代表前面的字符必须至少出现一次。（1次或多次）。例如，goo+gle可以匹配google、gooogle、goooogle等;
> `?`代表前面的字符最多只可以出现一次。（0次或1次）。例如，colou?r可以匹配color或者colour;
> `*`代表前面的字符可以不出现，也可以出现一次或者多次。（0次、1次或多次）。例如，0*42可以匹配42、042、0042、00042等。

- 匹配：圆括号()可以用来定义操作符的范围和优先度。例如，gr(a|e)y等价于gray|grey，(grand)?father匹配father和grandfather。上述这些构造子都可以自由组合，因此H(ae?|ä)ndel和H(a|ae|ä)ndel是相同的，表示{"Handel", "Haendel", "Händel"}。

&emsp;&emsp;表达式全集

![查看](/regex-md/regex.png)

&emsp;&emsp;元字符

![查看](/regex-md/regex-echar.png)

&emsp;&emsp;重复

![查看](/regex-md/regex-regex-repetition.png)

&emsp;&emsp;优先权

![查看](/regex-md/regex_order.png)

# java正则表达式

&emsp;&emsp;java.util.regex是一个用正则表达式所订制的模式来对字符串进行匹配工作的类库包。它包括两个类：Pattern和Matcher，Pattern：一个Pattern是一个正则表达式经编译后的表现模式。 Matcher：一个Matcher对象是一个状态机器，它依据Pattern对象做为匹配模式对字符串展开匹配检查。 首先一个Pattern实例订制了一个所用语法与PERL的类似的正则表达式经编译后的模式，然后一个Matcher实例在这个给定的Pattern实例的模式控制下进行字符串的匹配工作。

## 匹配模式

- 构建匹配模式

&emsp;&emsp;Pattern类用于创建一个正则表达式,也可以说创建一个匹配模式,它的构造方法是私有的,不可以直接创建,但可以通过Pattern.complie(String regex)简单工厂方法创建一个正则表达式。

```java
Pattern pattern = Pattern.compile("r+");
logger.info(pattern.pattern());
```

&emsp;&emsp;通过调用静态方法compile创建一个正则表达式的对象，在这个对象上调用pattern/toString方法返回这个正则表达式的字符串表现形式。

- 拆分给定序列

```java
Pattern pattern = Pattern.compile("\\.+?");

logger.info(Arrays.toString(pattern.split("https://ferry.org.cn")));
logger.info(Arrays.toString(pattern.split("https://ferry.org.cn", 4)));
logger.info(Arrays.toString(pattern.split("https://ferry.org.cn", 3)));
logger.info(Arrays.toString(pattern.split("https://ferry.org.cn", 2)));
logger.info(Arrays.toString(pattern.split("https://ferry.org.cn", 1)));
logger.info(Arrays.toString(pattern.split("https://ferry.org.cn", 0)));
logger.info(Arrays.toString(pattern.split("https://ferry.org.cn", -1)));
```

![查看](/regex-md/regex-split.png)

&emsp;&emsp;字符串的split方法，当给定的正则表达式不是简单字符时，调用pattern.split得到结果。

- 匹配给定输入

```java
// '.'匹配任意字符(除了\n和\r),'+'匹配前面的表达式一次或者多次
logger.info("{}", Pattern.matches(".+", "https://ferry.org.cn"));
```

&emsp;&emsp;本质上是创建匹配器，然后调用匹配器的matches方法。String的matches方法就是调用的Pattern.matches。

## 匹配器

- 创建匹配器

```java
Pattern pattern = Pattern.compile("\\.+?");
Matcher matcher = pattern.matcher("https://ferry.org.cn");
```

- 匹配操作

```java
// 匹配一个或者多个'.'
Pattern pattern = Pattern.compile("\\.+?");
Matcher matcher = pattern.matcher("https://ferry.org.cn");
// 整个输入序列是否完全匹配该正则表达式
logger.info("{}", matcher.matches());
// 从输入序列的开头开始匹配该正则表达式
logger.info("{}", matcher.lookingAt());
// 输入序列中是否有某一个自序列能够匹配正则表达式
logger.info("{}", matcher.find());
```

&emsp;&emsp;matches和lookingAt方法每次调用都会从输入序列的开头开始匹配，而find方法第一次从开头开始匹配，后面每次调用都是从上次匹配到的子字符串在输入序列的结束位置开始继续向后匹配。

- 获取匹配的子字符串在字符串中的索引

&emsp;&emsp;当使用matches,lookingAt或者find方法返回true时，此时就可以使用start和end查找匹配到的子字符串在字符串的索引位置。返回的索引数值总是等于下一个字符在字符串中的下标。

```java
Pattern pattern = Pattern.compile(".+");
Matcher matcher = pattern.matcher("https://ferry.org.cn");
if(matcher.matches()){
    logger.info("matches start: {}", matcher.start());
    logger.info("matches end: {}", matcher.end());
}
```

```java
Pattern pattern = Pattern.compile("^http(s)?");
Matcher matcher = pattern.matcher("https://ferry.org.cn");
if(matcher.lookingAt()){
    logger.info("lookingAt start: {}", matcher.start());
    logger.info("lookingAt end: {}", matcher.end());
}
```

```java
Pattern pattern = Pattern.compile("\\.+?");
Matcher matcher = pattern.matcher("https://ferry.org.cn");
int count = 0;
while (matcher.find()){
    logger.info("find {} start: {}", ++count, matcher.start());
    logger.info("find {} end: {}", count, matcher.end());
}
```

- 替换

```java
Pattern pattern = Pattern.compile("/+?");
Matcher matcher = pattern.matcher("ferry/org/cn");
logger.info("replaceFirst: {}", matcher.replaceFirst("."));
logger.info("replaceAll: {}", matcher.replaceAll("."));
```

- 分组

```java
Pattern pattern = Pattern.compile("(\\w+)\\.+?(\\w+)");
Matcher matcher = pattern.matcher("ferrysy.com");
if(matcher.find()){
    logger.info("group count: {}", matcher.groupCount());
    logger.info("group 0: {}", matcher.group(0));
    int i = 0;
    do{
        logger.info("group {}: {}", ++i, matcher.group(i));
    }while (i < matcher.groupCount());
}
```

![查看](/regex-md/regex-group.png)

&emsp;&emsp;正则表达式中，匹配`(pattern)`但并非是零宽预言的子字符串就是一个分组，可以使用groupCount方法查看分组数量，matcher.group(0)表示整个匹配的字符串，matcher.group(i)表示获取匹配的第i个分组。

# 进阶用法

## 分组

&emsp;&emsp;分组，用小括号来指定子表达式(也叫做分组)，可以指定这个子表达式的重复次数，你也可以对子表达式进行其它一些操作。

> 默认情况下，每个分组都有一个组号，规则是：从左向右，以分组的左括号为标志，第一个出现的分组的组号为 1，第二个为 2，以此类推；
> 后向引用用于重复搜索前面某个分组匹配的文本。例如，\1 代表分组1匹配的文本。注意，\1 代表的是分组1匹配到的文本，而不是分组1的表达式，其次，\1并不是分组；
> 可以指定子表达式的组名,语法：(?<groupName>regex)，其中，groupName即为分组名，<>也可以替换成''。

```java
Pattern pattern = Pattern.compile("(\\w+)\\.+?(\\w+)");
Matcher matcher = pattern.matcher("ferrysy.com");
if(matcher.find()){
    logger.info("group count: {}", matcher.groupCount());
    logger.info("group 0: {}", matcher.group(0));
    int i = 0;
    do{
        logger.info("group {}: {}", ++i, matcher.group(i));
    }while (i < matcher.groupCount());
}
```

```java
Pattern pattern = Pattern.compile("\\b(\\w+)\\b\\s\\1");
Matcher matcher = pattern.matcher("ferry ferry");
logger.info("{}", matcher.groupCount());
if(matcher.find()){
    logger.info(matcher.group(1));
}
```

```java
Pattern pattern = Pattern.compile("\\b(?<name>\\w+)\\b\\s\\1");
Matcher matcher = pattern.matcher("ferry ferry");
logger.info("{}", matcher.groupCount());
if(matcher.find()){
    logger.info(matcher.group(1));
    logger.info(matcher.group("name"));
}
```

## 零宽断言

&emsp;&emsp;查找在某些内容(但并不包括这些内容)之前或之后的东西，用于指定一个位置，这个位置应该满足一定的条件(即断言)，因此它们也被称为零宽断言。

- **零宽度正预测先行断言(?=exp)**

&emsp;&emsp;它断言自身出现的位置的后面能匹配表达式。

```java
// 可以匹配does不可以匹配do
Pattern pattern = Pattern.compile("do(?=es)");
Matcher matcher = pattern.matcher("do");
logger.info("{}", matcher.lookingAt());
matcher.reset("does");
logger.info("{}", matcher.lookingAt());
```

```java
// 限制字符串长度
Pattern pattern = Pattern.compile("(?=(^.{5}$))\\w+");
Matcher matcher = pattern.matcher("ferry");
logger.info("{}", matcher.matches());
```

- **零宽度正回顾后发断言(?<=exp)**

&emsp;&emsp;断言自身出现的位置的前面能匹配表达式 exp。

```java
// 可以匹配where不可以匹配here
Pattern pattern = Pattern.compile("(?<=wh)ere");
Matcher matcher = pattern.matcher("here");
logger.info("{}", matcher.find());
matcher.reset("where");
logger.info("{}", matcher.find());
```

- **零宽度负预测先行断言(?!exp)**

&emsp;&emsp;断言此位置的后面不能匹配表达式 exp。

```java
// 每五个字符后的字符不能是数字
Pattern pattern = Pattern.compile("(.{5}(?!\\d))+");
Matcher matcher = pattern.matcher("ferrylove2");
logger.info("{}", matcher.matches());
```

- **零宽度负回顾后发断言(?<!exp)**

&emsp;&emsp;断言此位置的前面不能匹配表达式exp。

```java
// 匹配五个下写字母且前面一个字符不是数字的
Pattern pattern = Pattern.compile("(?<!\\d)[a-z]{5}");
Matcher matcher = pattern.matcher("ferry");
logger.info("{}", matcher.matches());
matcher.reset("1ferry");
logger.info("{}", matcher.find());
```

## 贪婪和懒惰

- **贪婪**

&emsp;&emsp;当正则表达式中包含能接受重复的限定符时，通常的行为是（在使整个表达式能得到匹配的前提下）匹配尽可能多的字符。这被称为贪婪匹配。

```java
// 贪婪匹配
Pattern pattern = Pattern.compile("\\d.*\\d"); // 匹配两个数字，数字中间有零或多个字符
Matcher matcher = pattern.matcher("1fer22ry9");
while (matcher.find()){
    logger.info("{}", matcher.group());
}
// 最终匹配的是整个字符串
```

- **懒惰**

&emsp;&emsp;有时，我们更需要懒惰匹配，也就是匹配尽可能少的字符。前面给出的限定符都可以被转化为懒惰匹配模式，只要在它后面加上一个问号?。这样.*?就意味着匹配任意数量的重复，但是在能使整个匹配成功的前提下使用最少的重复。

```java
// 懒惰匹配
Pattern pattern = Pattern.compile("\\d.*?\\d"); // 匹配两个数字，数字中间有零或多个字符
Matcher matcher = pattern.matcher("11fer22ry99");
while (matcher.find()){
    logger.info("{}", matcher.group());
}
// 匹配的结果有三个，11，22，99
```

- **The match that begins earliest wins**

```java
Pattern pattern = Pattern.compile("\\d.*?\\d"); // 匹配两个数字，数字中间有零或多个字符
Matcher matcher = pattern.matcher("1fer22ry99");
while (matcher.find()){
    logger.info("{}", matcher.group());
}
// 匹配结果有两个，1fer2,2ry9
```

&emsp;&emsp;最先开始的匹配拥有最高优先权。上述正则表达式虽然是懒惰模式，理论上应该匹配22，99，但是第一个出现的数字是1，所以1具有优先匹配权。

- **懒惰限定符**

![查看](/regex-md/regex-lazy.png)

# 练习

&emsp;&emsp;正则表达式的学习最好是结合例子，单纯的记忆是很难玩的懂的，前面介绍了正则的java使用，那么在实践中会使用java的类去展示正则表达式的实际效果。

## 英文域名

&emsp;&emsp;域名的匹配规则如下：

> 域名由多个标签(各级域名)组成，每个标签长度限制63，域名总长度253；
> 各个标签之间用`.`连接；
> 标签可以由英文字母，阿拉伯数字和中横线组成
> 中横线不能连续出现、不能单独注册，也不能放在开头和结尾。

```java

```

### 邮箱校验

&emsp;&emsp;先说一下邮箱的规则，邮箱分两部分有符号`@`连接，第一部分：组成元素可以是大小写英文字母，中横线，阿拉伯数字，下划线，英文句号，这里我们暂且规定邮箱第一部分长度不小于5，且只能用大小写英文字母开头；第二部分：域名。







