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

- 为快速查找而设计的`Set`，存入`HashSet`的元素必须定义`hashCode()`。`HashSet`存储元素的顺序是按照底层哈希桶号从小到大排序，如果桶号相同就按照放入桶的先后顺序。

### TreeSet

- 保持次序的`Set`，底层为树结构。使用它可以从`Set`中提取有序的序列。元素必须实现`Comparable`接口

### LinkedHashSet

- 具有`HashSet`的查询速度，且内部使用链表维护元素的顺序（插入的顺序）。

### SortedSet

- 可以保证其中的元素处于排序状态，并提供一些附加功能

## 队列

### LinkedList

- 队列的基本实现。

### 优先级队列

- `PriorityQueue`：按照元素从小到大排列。

## Map

### Map的各种实现

- `HashMap`：`Map`基于散列表的实现。插入和查询键值对的开销是固定的，可以通过构造器设置容量和负载因子，以调整容器性能

> 负载因子：如果容器持有元素超过一定比例会进行扩容，此时作为底层实现的旧数组会被更大容量的新数组取代。这个比例就是负载因子。

- `LinkedHashMap`：类似于`HashMap`，但是迭代遍历时，取得键值对的顺序是插入顺序，或者是最近最少使用（LRU）次序。仅比`HashMap`慢一点，而在迭代访问时反而更快，因为其使用链表维护内部次序。
- `TreeMap`：基于红黑树的实现。查看键或者键值对时，它们会被排序。`TreeMap`的特点在于，所得到的结果是经过排序的（按元素大小排序）。`TreeMap`是唯一带有`subMap()`方法的`Map`，它可以返回一棵子树。
- `WeakHashMap`：弱键映射，允许释放映射所指向的对象，这是为解决某类特殊问题而设计的。如果映射之外没有引用指向某个键，则此键可以被垃圾回收。
- `ConcurrentHashMap`：一种线程安全的`Map`，它不涉及同步加锁。
- `IdentityHashMap`：使用`==`代替`equals()`对键进行比较的散列映射，专为解决特殊问题而设计的。

### 散列

- `HashMap`使用了散列码作为键的搜索。在Java中，`hashCode()`是根类`Object`的方法，因此所有Java对象都能产生散列码。使用`hashCode()`进行查询能够显著地提高性能。
- 默认的`Object#equals()`只是比较对象地址，如果需要某个自定义的类作为键使用，需要重载`hashCode()`和`equals()`。

## 容器的选择

### List

- `ArrayList`应该作为默认选择，只有当程序的性能因为经常从表中间进行插入和删除而变差的时候，才去选择`LinkedList`。如果使用的是固定数量的元素，也可以直接使用数组。

### Set

- `HashSet`的性能基本上总是比`TreeSet`好，特别是在添加和查询元素时。当需要对元素进行排序，或者需要对元素进行迭代时才会需要`TreeSet`。

### Map

- 类似于`Set`的结果，`TreeMap`比`HashMap`要慢。

## 实用方法

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `max(Collection)`, `min(Collection)`                         | 返回参数`Collection`中最大或者最小的元素，采用`Collection`中内置的自然比较法 |
| `max(Collection, Comparator)`, `min(Collection, Comparator)` | 返回参数`Collection`中最大或最小的元素，采用参数中的`Comparator`进行比较 |
| `indexOfSubList(List source, List target)`                   | 返回`target`在`source`中第一次出现的位置，或者在找不到时返回`-1` |
| `lastIndexOfSubList(List source, List target)`               | 返回`target`在`source`中最后一次出现的位置，或者在找不到时返回`-1` |
| `replaceAll(List<T>, T oldVal, T newVal)`                    | 使用`newVal`替换所有`oldVal`                                 |
| `reverse(List)`                                              | 逆转所有元素的次序                                           |
| `reverseOrder()`, `reverseOrder(Comparator<T>)`              | 返回一个`Comparator`，它可以以逆转实现了`Comparator<T>`的对象集合的自然顺序。第二个版本可以逆转所提供的`Comparator`的顺序 |
| `rotate(List, int distance)`                                 | 所有元素向后移动`distance`个位置，将末尾的元素循环到前面来   |
| `shuffle(List)`, `shuffle(List, Random)`                     | 随机改变指定列表顺序。第一种形式提供其自己的随机机制，第二种则来源于参数`Random`对象 |
| `sort(List<T>)`, `sort(List<T>, Comparator<? super T> c)`    | 使用`List<T>`中的自然顺序排序。第二种形式允许提供用于排序的`Comparator` |
| `copy(List<? super T> dest, List<? extends T> src)`          | 将`src`中的元素复制到`dest`                                  |
| `swap(List, int i, int j)`                                   | 交换`list`中位置`i`与位置`j`的元素，通常比手动实现要快       |
| `fill(LIst<? super T>, T x)`                                 | 用对象`x`替换`list`中的所有元素                              |
| `disjoint(Collection, Collection)`                           | 当两个集合没有重复元素时返回`true`                           |
| `frequency(Collection, Object x)`                            | 返回`Collection`中`x`出现的次数                              |
| `emptyList()`, `emptyMap()`, `emptySet()`                    | 返回不可变且为泛型的`List`、`Map`、`Set`                     |
| `singleton(T x)`, `singletonList(T x)`, `singletonMap(K key, V value)` | 产生不可变的`Set<T>`、`List<T>`、`Map<K, V>`。               |

- 在比较字符串而使用`max()`、`min()`、`sort()`等方法时，使用参数`String.CASE_INSENSITIVE_ORDER`可以忽略大小写。

### 同步控制

- `Collections.synchonizedCollection()`可以传入一个新生成的容器，返回的是有同步功能的版本。类似的还有`Collections.synchonizedList()`、`Collections.synchonizedSet()`、`Collections.synchonizedMap()`等。
- 快速报错：Java容器的一种保护机制，能够防止多个进程同时修改同一个容器的内容。它会探查容器上的任何除了当前进程进行的操作以外的所有变化，一旦发现其它进行修改了容器，就立刻抛出`ConcurrentModificationException`异常。