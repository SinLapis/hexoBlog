---
title: MySQL技术内幕笔记0x01
date: 2019-09-02 08:40:34
categories: 数据库
tags:
  - MySQL
  - InnoDB
  - 表
  - 视图
---

# 表（二）

## 视图

- 在MySQL数据库中，视图是一个命名的虚表，它由一个SQL查询来定义，可以当作表使用。与持久表不同的是，视图中的入局没有实际的物理存储。

### 视图的作用

- 视图的主要用途之一是被用作一个抽象装置，特别是对于一些应用程序，程序本身不需要关心基表的结构，只需要按照视图的定义来取数据或更新数据，因此，视图同时在一定程度上起到一个安全层的作用。

- 用户可以对某些视图进行更新操作，一般称可以进行更新操作的视图为可更新视图。如果加上with check option选项，MySQL数据库会对更新视图插入的数据进行检查，对于不满足视图定义条件的插入会抛出一个异常，不允许视图中数据更新。

## 分区表

- 分区的过程是将一个表或索引分解为多个更小、更可管理的部分。就访问数据库而言，从逻辑上讲，只有一个表或一个索引，但是在物理上这个表或索引可能由数十个物理分区组成。每个分区都是独立的对象，可以独自处理，也可以作为一个更大的对象的一部分进行处理。
- 水平分区是指将同一表中不同行的记录分配到不同的物理文件中，垂直分区是指将同一表中不同列的记录分配到不同的物理文件中。MySQL数据库支持的分区类型为水平分区，不支持垂直分区。
- 局部分区索引是指一个分区中既存放了数据又存放了索引。与之相对的是全局分区，数据存放在各个分区中，但是所有数据的索引放在一个对象中。MySQL的分区是局部分区索引。
- 分区主要用于数据库高可用性的管理。
- 不论创建何种类型的分区，如果表中存在主键或唯一索引时，分区列必须是唯一索引的一个组成部分。如果建表时没有指定主键，唯一索引，可以指定任何一个列为分区列。

### 分区类型

#### range分区

- 基于属于一个给定连续区间的列值对行数据进行分区。

```sql
create table t (
id int
)engine=innodb
partition by range(id) (
partition p0 values less than (10),
partition p1 values less than (20)
);
```

#### list分区

- 同range分区，只是list分区面向的是离散值。如果插入的值不在分区定义中，MySQL数据库会抛出异常。

```sql
create table t (
a int,
b int
)engine=innodb
partition by list(b) (
partition p0 values in (1, 3, 5, 7, 9),
partition p1 values in (0, 2, 4, 6, 8)
);
```

#### hash分区

- 根据用户自定义的表达式的返回值来进行分区，返回值不能为负数。此外还需要指定分区数量。

```sql
create table t (
a int,
b datetime
)engine=innodb
partition by hash (year(b))
partitions 4;
```

#### key分区

- 使用MySQL数据库提供的函数进行分区。

```sql
create table t (
a int,
b datetime
)engine=innodb
partition by key(b)
partitions 4;
```

#### columns分区

- 可以直接使用非整型的数据进行分区，分区根据类型直接比较而得。支持的类型有所有整数类型（int、smallint、tinyint、bigint），日期类型（date、datetime），字符串类型（char、varchar、binary、varbinary）。

```sql
create table t (
a int,
b datetime
)engine=innodb
partition by range columns(b) (
partition p0 values less than('2009-01-01'),
partition p1 values less than('2010-01-01')
);
```

### 子分区

- 子分区是在分区的基础上再进行分区，有时也称这种分区为复合分区。MySQL数据库允许在range和list分区上再进行hash或key分区。

```sql
create table ts (
    a int, 
    b date
)engine=innodb
partition by range(year(b))
subpartition by hash(to_days(b))
subpartitions 2 (
    partition p0 values less than (1990),
    partition p1 values less than (2000),
    partition p2 values less than maxvalue
);
```

### 分区中的null值

- MySQL数据库总是认为null值小于任何一个非null值。
- 对于range分区，如果向分区列插入了null值，则MySQL数据库会将该值放入最左边的分区。
- 在list分区中使用null值，必须显式指定在哪个分区放入null值，否则会报错。
- hash和key分区的任何分区函数都会将含有null值的记录都返回0。