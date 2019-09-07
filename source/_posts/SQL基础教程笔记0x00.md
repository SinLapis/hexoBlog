---
title: SQL基础教程笔记0x00
date: 2019-09-07 10:05:05
categories: SQL
tags:
  - SQL
  - 聚合查询
---

# 查询基础

## 逻辑运算符

### 含有NULL时的真值

- SQL中的逻辑运算是三值逻辑，即包括真、假、不确定。

# 聚合与排序

## 为聚合结果指定条件

### having子句

- having子句用于对集合指定条件过滤。having子句位于group by子句后面。

```sql
select `product_type`, count(*)
from shop
group by `product_type`
having count(*) = 2;
```

- having子句中能够使用的要素有：常数、聚合函数、group by子句中指定的列名（即聚合键）。

# 复杂查询

## 视图

### 视图的创建

```sql
create view shop_sum (`product_type`, `cnt_product`)
as
select  `product_type`, count(*)
from shop
group by `product_type`;
```

### 限制

- 定义视图时不能使用`order by`。
- 对视图更新是有条件的：
  1. `select`子句中未使用`distinct`。
  2. `from`子句中只有一张表。
  3. 未使用`group by`子句。
  4. 未使用`having`子句。

## 子查询

- 子查询就是将用来定义视图的select子句直接用于from子句中。

```sql
select `product_type`, `cnt_product`
from (
         select `product_type`, count(*) as `cnt_product`
         from `shop`
         group by `product_type`
     ) as `product_sum`;
```

### 标量子查询

- 标量子查询必须而且只能返回1行1列的结果。

```sql
/* where子句中无法使用聚合函数，可以使用标量子查询代替。*/
select `product_id`, `product_name`, `sale_price`
from `shop`
where sale_price > (
    select avg(sale_price)
    from `shop`
    );
```

## 关联子查询

- 在普通子查询中添加where子句，使其变为标量子查询。

```sql
select `product_id`, `product_name`, `sale_price`
from `shop` as `s1`
where sale_price > (
    select avg(sale_price)
    from `shop` as `s2`
    where `s1`.`product_type` = `s2`.`product_type`
    group by `product_type`
    )
;
```

# 函数、谓词、case表达式

## 谓词

### like

- `%`代表0字符及以上的任意字符串，`_`代表任意一个字符。

```sql
/* pg */
select *
from test.like_sample.like_test
where strcol like 'abc%';

select *
from test.like_sample.like_test
where strcol like 'abc__';
```

### between

- 范围查询，包括两个边界。

```sql
select *
from `shop`
where sale_price between 100 and 1000;
```

### is null、is not null

- 判断是否为`null`。

### in

- `or`的简便写法。

```sql
select `product_name`, `purchase_price`
from `shop`
where putchase_price in (320, 500, 5000);
```

### 使用子查询作为in的参数

```sql
select `product_name`, `sale_price
```



