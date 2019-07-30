---
title: Java8实战笔记0x01
date: 2019-07-30 14:22:58
categories: Java
tags:
  - Java
  - 笔记
  - 流
---

# 流

## 流简介

- 流是从支持数据处理操作的源生成的元素序列。
- 集合的核心是数据，而流的目的在于表达计算。
- 和迭代器类似，流只能遍历一次。
- 使用`Collection`接口需要用户做迭代，称为外部迭代。而使用`Stream`则代替用户做了迭代，称为内部迭代。

## 流操作

```java
List<String> names = menu.stream()
                         // 中间操作
                         .filter(d -> d.getCalories())
                         .map(Dist::getName)
                         .limit(3)
                         // 终端操作
                         .collect(toList());
```

- 诸如`filter`或`sorted`等中间操作会返回另一个流，这让多个操作可以连接起来形成一个查询。除非流水线上触发一个终端操作，否则中间操作不会执行任何处理，因为中间操作一般都可以合并起来，在终端操作时一次性全部处理，这种技术称作循环合并。
- 终端操作会从流的流水线生成结果。其结果是任何不是流的值，比如`List`、`Integer`，甚至是`void`。