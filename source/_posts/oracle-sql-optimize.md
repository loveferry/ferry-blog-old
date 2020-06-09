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

&emsp;&emsp;基数/表的总行数*100%。

&emsp;&emsp;什么样的列必须创建索引？当一个列出现在 where 条件中，该列没有创建索引并且选择性大于20%，那么该列就必须创建索引。

```sql
select count(distinct t.sex) / count(*) from user t
```

# 全自动优化脚本

## 抓住必须创建索引的列


- 刷行数据库监控信息

```sql
begin
  dbms_stats.flush_database_monitoring_info;
end;
```

- 查询指定用户指定表出现在where条件中的列信息

```sql
select r.name as owner,
       o.name as table_name,
       c.name as column_name,
       u.equality_preds,     -- 等值过滤
       u.equijoin_preds,     -- 等值join，ex: where a.id=b.id
       u.nonequijoin_preds,  -- 不等于join
       u.range_preds,        -- 返回过滤次数
       u.like_preds,         -- like过滤
       u.null_preds,         -- null过滤
       u.timestamp
from sys.col_usage$ u,
     sys.obj$ o,
     sys.col$ c,
     sys.user$ r
where u.obj# = o.obj#
  and u.obj# = c.obj#
  and u.intcol# = c.col#
  and r.name = 'RETAIL'
  and o.name = 'CON_CONTRACT'
```

- 查询指定用户指定表的索引信息

```sql
select c.table_owner, c.table_name, c.column_name, c.index_name
from dba_ind_columns c
where c.table_owner = 'RETAIL'
  and c.table_name = 'CON_CONTRACT';
```

- 指定用户指定表的选择性大于20%的列

```sql
select a.owner, a.table_name, a.column_name, round(a.num_distinct / b.num_rows * 100, 2) selectivity
from dba_tab_col_statistics a,
     dba_tables b
where a.owner = b.owner
  and a.table_name = b.table_name
  and a.owner = 'RETAIL'
  and a.table_name = 'CON_CONTRACT'
  and a.num_distinct / b.num_rows >= 0.2;
```


- 最终的脚本

```sql
select t1.owner, t1.table_name, t1.column_name, t1.num_rows, t1.Cardinality, t1.selectivity, 'Need index' as notice
from (select b.owner,
             a.table_name,
             a.column_name,
             b.num_rows,
             a.num_distinct                              Cardinality,
             round(a.num_distinct / b.num_rows * 100, 2) selectivity
      from dba_tab_col_statistics a,
           dba_tables b
      where a.owner = b.owner
        and a.table_name = b.table_name
        and a.owner = 'RETAIL'
        and a.table_name = 'CON_CONTRACT') t1
where t1.selectivity >= 20
  and not exists(select 1
                 from dba_ind_columns t2
                 where t2.table_owner = t1.owner
                   and t2.table_name = t1.table_name
                   and t2.column_name = t1.column_name)
  and exists(select 1
             from sys.col_usage$ u,
                  sys.obj$ o,
                  sys.col$ c,
                  sys.user$ r
             where o.obj# = u.obj#
               and c.obj# = u.obj#
               and c.col# = u.intcol#
               and r.name = t1.owner
               and o.name = t1.table_name
               and c.name = t1.column_name);
```


