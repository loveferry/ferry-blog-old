---
title: oracle sql 调优
date: 2020-06-08 12:39:22
tags:
    - oracle
    - sql
categories: oracle
---

&emsp;&emsp;oracle数据库sql调优学习记录笔记。

<!-- more -->

# 核心思想

## 只有大表才会产生性能

# 基本概念及相关自动化脚本

## 基数

&emsp;&emsp;某个列唯一键的数量叫做基数。主键列的基数等于行数。性别列的基数等于2。

&emsp;&emsp;基数的高低影响列的数据分布。

&emsp;&emsp;当查询结果返回表中5%以内的数据时，应该走索引；当查询结果返回表中超过5%的数据时，应该走全表扫描。

- **查询性别列的基数**

```sql
select count(distinct t.sex) from user t 
```

## 选择性

&emsp;&emsp;基数/表的总行数*100%

```sql
select count(distinct t.sex) / count(*) from user t
```

