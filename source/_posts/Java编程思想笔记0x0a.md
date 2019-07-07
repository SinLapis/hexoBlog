---
title: Java编程思想笔记0x0a
date: 2019-07-07 16:52:29
categories: Java
tags:
  - Java
  - 笔记
---

# 容器深入研究

## Collection的功能方法

| 方法                                      | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| `boolean add(T)`                          | 添加元素，如果类型不为`T`则返回`false`。可选                 |
| `boolean addAll(Collection<? extends T>)` | 添加所有元素，只要添加了元素就返回`true`。可选               |
| `void clear()`                            | 移除所有元素。可选                                           |
| `boolean contains(T)`                     | 如果容器包含该元素，则返回`true`                             |
| `boolean containsAll(Collection<?>)`      | 如果容器中包含所有元素，则返回`true`                         |
| `boolean isEmpty()`                       | 容器中没有元素返回`true`                                     |
| `Iterator<T> iterator()`                  | 返回一个可以遍历容器所有元素的`Iterator<T>`                  |
| `boolean remove(Object)`                  | 如果元素在容器中，则移除一个该元素的实例。如果发生了移除则返回`true`。可选 |
| `boolean removeAll(Collection<?>)`        | 移除参数中所有的元素，如果发生了移除就返回`true`。可选       |
| `boolean retainAll(Collection<?>)`        | 只保存参数中的元素，只要`Collection`发生了改变就返回`true`   |
| `int size()`                              | 返回容器中元素的数目                                         |
| `Object[] toArray()`                      | 返回包含容器中所有元素的数组                                 |
| `<T> T[] toArray(T[] a)`                  | 返回包含容器中所有元素的数组，数组类型与参数类型一致         |

## 可选操作

- 在`Collection`方法中，一些方法是可选的，这意味着继承`Collection`时这些方法可以不进行覆盖。此时导出类对象调用该方法时会出现`UnsupportedOperationException`。`Collection`的可选方法实现如下:

  ```java
  public boolean add(E e) {
      throw new UnsupportedOperationException();
  }
  ```

  可见，如果导出类不实现可选方法，那么该方法会调用`Collection`中的实现，即抛出一个`UnsupportedOperationException`异常。

## Set与存储顺序

### Set

- 存入`Set`的每个元素都必须是唯一的，因为`Set`不保存重复元素。加入`Set`的元素必须定义`equals()`方法一确保对象的唯一性。`Set`和`Collection`有完全一样的接口。`Set`不保证维护元素的次序。

### HashSet

- 为快速查找而设计的`Set`，存入`HashSet`的元素必须定义`hashCode()`。

### TreeSet

- 保持次序的`Set`，底层为树结构。使用它可以从`Set`中提取有序的序列。元素必须实现`Comparable`接口

### LinkedHashSet

- 具有`HashSet`的查询速度，且内部使用链表维护元素的顺序（插入的顺序）。