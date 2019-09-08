---
title: SQL基础教程笔记0x01
date: 2019-09-08 09:18:25
categories: SQL
tags:
  - SQL
---

# 集合运算

## 表的加减法

### union

- 对表进行并集操作。可以使用`union all`使结果包含重复行。

```sql
select product_id, product_name
from product
union
select product_id, product_name
from product_2;
```

| product\_id | product\_name |
| :---------- | :------------ |
| 0001        | T恤           |
| 0002        | 打孔器        |
| 0003        | 运动T恤       |
| 0004        | 菜刀          |
| 0005        | 高压锅        |
| 0006        | 叉子          |
| 0007        | 擦菜板        |
| 0008        | 圆珠笔        |
| 0009        | 手套          |
| 0010        | 水壶          |

- 注意：作为`union`的运算对象的记录的列数必须相同，列的类型必须一致；`order by`只能在最后使用一次。

### intersect

- 对表进行交集操作。MySQL不支持该操作。

```sql
/* postgreSQL */
select public.product.product_id, public.product.product_name
from public.product
intersect
select public.product_2.product_id, public.product_2.product_name
from public.product_2;
```

| product\_id | product\_name |
| :---------- | :------------ |
| 0001        | T恤           |
| 0002        | 打孔器        |
| 0003        | 运动T恤       |

```sql
/* MySQL用exists模拟 */
select product_id, product_name
from product as p
where exists(
              select product_id
              from product_2 as p2
              where p.product_id = p2.product_id
          );
```

### except

- 对表进行差集操作，注意该操作不满足交换律。

```sql
/* postgreSQL */
select public.product.product_id, public.product.product_name
from public.product
except
select public.product_2.product_id, public.product_2.product_name
from public.product_2
```

| product\_id | product\_name |
| :---------- | :------------ |
| 0004        | 菜刀          |
| 0005        | 高压锅        |
| 0006        | 叉子          |
| 0007        | 擦菜板        |
| 0008        | 圆珠笔        |

```sql
/* MySQL用not exists模拟 */
select product_id, product_name
from product as p
where not exists(
              select product_id
              from product_2 as p2
              where p.product_id = p2.product_id
          );
```

| product\_id | product\_name |
| :---------- | :------------ |
| 0004        | 菜刀          |
| 0005        | 高压锅        |
| 0008        | 圆珠笔        |
| 0006        | 叉子          |
| 0007        | 擦菜板        |

## 联结

### 内联结

- 以一张表中的列为桥梁，将其他表中满足同样条件的列汇集到同一结果之中。

```sql
select sp.shop_id, sp.shop_name, sp.product_id, p.product_name, p.sale_price
from public.shop_product as sp inner join public.product as p
on sp.product_id = p.product_id
```

| shop\_id | shop\_name | product\_id | product\_name | sale\_price |
| :------- | :--------- | :---------- | :------------ | :---------- |
| 000D     | 福冈       | 0001        | T恤           | 1000        |
| 000A     | 东京       | 0001        | T恤           | 1000        |
| 000B     | 名古屋     | 0002        | 打孔器        | 500         |
| 000A     | 东京       | 0002        | 打孔器        | 500         |
| 000C     | 大阪       | 0003        | 运动T恤       | 4000        |
| 000B     | 名古屋     | 0003        | 运动T恤       | 4000        |
| 000A     | 东京       | 0003        | 运动T恤       | 4000        |
| 000C     | 大阪       | 0004        | 菜刀          | 3000        |
| 000B     | 名古屋     | 0004        | 菜刀          | 3000        |
| 000C     | 大阪       | 0006        | 叉子          | 500         |
| 000B     | 名古屋     | 0006        | 叉子          | 500         |
| 000C     | 大阪       | 0007        | 擦菜板        | 880         |
| 000B     | 名古屋     | 0007        | 擦菜板        | 880         |

### 外联结

- 和内联结基本相同，但是会选择主表的全部信息（见下面示例）。

```sql
select sp.shop_id, sp.shop_name, sp.product_id, p.product_name, p.sale_price
from public.shop_product as sp right outer join public.product as p
on sp.product_id = p.product_id
```

| shop\_id | shop\_name | product\_id | product\_name | sale\_price |
| :------- | :--------- | :---------- | :------------ | :---------- |
| 000D     | 福冈       | 0001        | T恤           | 1000        |
| 000A     | 东京       | 0001        | T恤           | 1000        |
| 000B     | 名古屋     | 0002        | 打孔器        | 500         |
| 000A     | 东京       | 0002        | 打孔器        | 500         |
| 000C     | 大阪       | 0003        | 运动T恤       | 4000        |
| 000B     | 名古屋     | 0003        | 运动T恤       | 4000        |
| 000A     | 东京       | 0003        | 运动T恤       | 4000        |
| 000C     | 大阪       | 0004        | 菜刀          | 3000        |
| 000B     | 名古屋     | 0004        | 菜刀          | 3000        |
| NULL     | NULL       | NULL        | 高压锅        | 6800        |
| 000C     | 大阪       | 0006        | 叉子          | 500         |
| 000B     | 名古屋     | 0006        | 叉子          | 500         |
| 000C     | 大阪       | 0007        | 擦菜板        | 880         |
| 000B     | 名古屋     | 0007        | 擦菜板        | 880         |
| NULL     | NULL       | NULL        | 圆珠笔        | 100         |

有两条记录另一张表上没有。

```sql
select sp.shop_id, sp.shop_name, sp.product_id, p.product_name, p.sale_price
from public.shop_product as sp left join public.product as p
on sp.product_id = p.product_id
```

| shop\_id | shop\_name | product\_id | product\_name | sale\_price |
| :------- | :--------- | :---------- | :------------ | :---------- |
| 000D     | 福冈       | 0001        | T恤           | 1000        |
| 000A     | 东京       | 0001        | T恤           | 1000        |
| 000B     | 名古屋     | 0002        | 打孔器        | 500         |
| 000A     | 东京       | 0002        | 打孔器        | 500         |
| 000C     | 大阪       | 0003        | 运动T恤       | 4000        |
| 000B     | 名古屋     | 0003        | 运动T恤       | 4000        |
| 000A     | 东京       | 0003        | 运动T恤       | 4000        |
| 000C     | 大阪       | 0004        | 菜刀          | 3000        |
| 000B     | 名古屋     | 0004        | 菜刀          | 3000        |
| 000C     | 大阪       | 0006        | 叉子          | 500         |
| 000B     | 名古屋     | 0006        | 叉子          | 500         |
| 000C     | 大阪       | 0007        | 擦菜板        | 880         |
| 000B     | 名古屋     | 0007        | 擦菜板        | 880         |