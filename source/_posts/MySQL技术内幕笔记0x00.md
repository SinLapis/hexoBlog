---
title: MySQL技术内幕笔记0x00
date: 2019-08-28 10:13:09
categories: 数据库
tags:
  - MySQL
  - InnoDB
  - 表
---

# 表（一）

## 索引组织表

- 在InnoDB存储引擎中，表都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表。如果在创建表时没有显式地指定主键，则InnoDB会按以下方式选择或创建主键：
  - 首先判断表中是否有非空的唯一索引，如果有则该列为主键。该方式选择的根据是定义索引的顺序，而不是建表时列的顺序。
  - 否则，InnoDB存储引擎会自动创建一个6字节大小的指针。

## InnoDB逻辑存储结构

- 从InnoDB存储引擎的逻辑存储结构看，所有数据都被逻辑地存放在一个空间中，称之为表空间。表空间又由段、区、页组成。
- 表空间可以看作是InnoDB存储引擎逻辑结构的最高层 ，所有数据都存放在表空间中。
- 常见的段有数据段、索引段、回滚段等。数据段即为B+树的叶子节点，索引段为B+树的非叶子节点。
- 区是由连续页组成的空间，在任何情况下每个区的大小都为1MB。为了保证区中页的连续性，InnoDB存储引擎一次从磁盘中申请4~5个区。在默认情况下，InnoDB存储引擎页大小为16KB，即一个区中一共有64个连续的页。
- 页是InnoDB磁盘管理的最小单位。

## 约束

### 数据完整性

- 关系性数据库和文件系统的一个不同点是，关系数据库本身能保证存储数据的完整性，不需要应用程序的控制，而文件系统一般需要在程序端进行控制。
- 一般来说，数据完整性有以下三种形式：
  - 实体完整性保证表中有一个主键，可以通过定义Primary Key或Unique Key约束或者定义一个触发器来保证实体的完整性。
  - 域完整性保证数据每列的值满足特定的条件，可以选择合适的数据类型确保一个入局值满足特定条件，使用外键约束，编写触发器，或者考虑DEFAULT约束。
  - 参照完整性保证两张表之间的关系。可以使用外键以强制参照完整性，也可以使用触发器。
- InnoDB存储引擎提供了以下几种约束：primary key、unique key、foreign key、default、not null

### 约束的创建

- 约束的创建有两种方式：
  - 表建立时就进行约束定义
  - 利用alter table命令来进行创建约束
- 对于主键约束而言，其默认约束名为primary。对于unique key约束而言，默认约束名和列名一样，也可以人为指定名字。

### 约束和索引的区别

- 当用户创建了一个唯一索引就创建了一个唯一的约束。约束偏向于一个逻辑概念，用来保证数据完整性，而索引是一个数据结构，既有逻辑上的概念，在数据库中还代表着物理存储方式。

### 对错误数据的约束

- 在某些魔神设置下，MySQL数据库允许非法的或者不正确的数据插入或更新，又或者可以在数据库内部将其转化为一个合法的值。

### 触发器约束

- 触发器的作用是在执行insert、delete、update命令之前或之后自动调用SQL命令或存储过程。最多可以为一个表建立6个触发器，即分别为insert、update、delete的before和after各定义一个。当前MySQL数据库只支持for each row的 触发方式，即按每行记录进行触发。
- 通过触发器，用户可以实现MySQL数据库本身并不支持的一些特性。

```sql
create table user_cash (
    user_id int not null ,
    cash int unsigned not null 
);

insert into `user_cash`(`user_id`, `cash`) value(1, 1000);
# 非正常行为
update `user_cash` set `cash` = `cash` - (-20) where `user_id` = 1;
```

```sql
# 添加触发器和错误日志
create table `user_cash_err_log` (
    user_id int not null ,
    old_cash int unsigned not null ,
    new_cash int unsigned not null ,
    user varchar(30),
    time datetime
);

create trigger tgr_user_cash_update before update on `user_cash`
    for each row
    begin
        if new.`cash` - old.`cash` > 0 then
            insert into `user_cash_err_log`
            select old.`user_id`, old.`cash`, new.`cash`, USER(), NOW();
            set new.`cash` = old.`cash`;
        end if;
    end;
```

```sql
# 再次尝试非正常行为
update `user_cash` set `cash` = `cash` - (-20) where `user_id` = 1;
select * from user_cash_err_log;
# 1	1020	1040	root@localhost	2019-08-29 10:06:27
select * from user_cash;
# 1	1020
```

### 外键约束

- 一般称被引用表为父表，引用的表称为子表。外键定义时的on delete和on update表示在对父表进行delete和update操作时，对子表所做的操作。
- 可定义的子表操包括cascade、set null、no action、restrict。cascade（级联）表示当父级发生delete或update操作时，对相应的子表中的数据也进行delete或update操作。set null表示父表发生delete或update操作时，相应的子表中的数据被更新为null值，但是子表中相对应的列必须允许为null值。no action和restrict在MySQL中等价，都表示父表发生delete或update操作时抛出错误，不允许这类操作发生。
