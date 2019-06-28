---
title: Java编程思想笔记0x02
date: 2019-06-28 19:36:43
categories: Java
tags:
  - Java
  - 笔记
---

# 持有对象

## 添加一组元素

- ```Collections.addAll()```方法接受一个```Collection```对象，以及一个数组或者一个逗号分隔符列表，将元素添加到```Collection```中。推荐使用此方法。
- ```Arrays.asList()```方法接受数组或列表，但是其返回的对象类型```List```并不是```java.util.List```，而是```Arrays```内的一个静态内部类，没有```add()```和```remove()```方法。

## List

- ```List```在```Collection```中添加了大量方法，可以在```List```中间插入和移除元素

### 类型

- ```ArrayList```：长于随机访问元素，但是在```List```中间插入和移除元素较慢
- ```LinkedList```：在中间插入和删除操作代价较低，提供了优化的顺序访问，但是随机访问相对较慢

### 部分方法

- `remove()`：此方法接受元素类型对象（调用其`equals()`方法）或者在`List`中的序号，来删除指定元素。注意如果`List`中元素为`Integer`类型时，删除值为`x`的方式为`remove((Integer) x)`而不是`remove(x)`，后者会删掉序号为`x`的元素。
- `subList()`：获取子列表
- `containsAll()`：是否包含参数序列中所有的值。与参数顺序无关。
- `retainsALL()`：参数为另一个`List`，求两个`List`交集，结果保存在调用对象里。

## LinkedList

- 实现了`List`的基本接口，添加了可以使其用作栈、队列或双端队列的方法。

## Stack

## Set

### 类型

- `HashSet`：使用散列函数实现，查找速度较快
- `TreeSet`：基于红黑树实现
- `LinkedHashSet`：同样使用散列函数实现，但同时用链表维护了元素插入顺序

## Map

## Queue

- `offer()`：在允许的情况下将一个元素插入队尾，或者返回`false`。
- `PriorityQueue`：使用`offer()`插入时会对元素进行排序。默认顺序是元素在队列中的自然顺序，可通过修改`Comparator`来修改顺序。

