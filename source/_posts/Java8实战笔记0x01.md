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

# 使用流

 ## 筛选和切片

### 用谓词筛选

- `filter`方法：接受一个谓词（返回`boolean`的函数）作为参数，并返回一个包括所有符合谓词的元素的流。

```java
List<Dish> vegetarianMenu = menu.stream()
                                .filter(Dish::isVegetarian)
                                .collect(toList());
```

### 提取不同元素

- `distinct`方法：无参数，返回一个元素各异的流（根据流所生成元素的`hashCode()`和`equals()`方法实现）。

```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
       .filter(i -> i % 2 == 0)
       .distinct()
       .forEach(System.out::println);
/* Output:
2
4
*/
```

### 截短流

- `limit(n)`方法：返回一个不超过给定长度`n`的流。如果流是有序的，则最多返回钱`n`个元素；如果流是无序的（例如`Set`），则返回结果不会以任何顺序排列。

### 跳过元素

- `skip(n)`方法：返回一个去掉前`n`个元素的流。如果流中的元素不足`n`个，则返回一个空流。与`limit(n)`是互补操作。

## 映射

### 对流中的每个元素应用函数

- `map`方法：接受一个函数作为参数，这个函数会被应用到每个元素上，并将其映射成一个新的元素（与转换的区别是，该步骤是创建一个新的版本而不是修改）。

```java
List<String> dishName = menu.stream()
                            .map(Dish::getName)
                            .collect(toList());
```

### 流的扁平化

- `flatMap`方法：传入的方法生成的流不会各自独立，而是会被合并，扁平化称为一个流。

## 查找和匹配

- `anyMatch`检查谓词是否至少匹配一个元素；`allMatch`会检查谓词是否匹配所有元素，相对的`noneMatch`则会确保流中没有任何元素与给定的谓词匹配。这三种方法都用到了短路。
- `findAny`可以返回流中的任意元素，而`findFirst`则返回有出现顺序的流中的第一个元素。使用时配合`Optional<T>`使用。此外`findAny`在使用并行流时限制较少，如果不关心返回哪个元素，那么应该使用`findAny`。

## 归约

```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
```

- 上面代码展示了用`reduce`求一组整数的和，`reduce`接受一个初始值和一个函数，它将使用函数反复结合每个元素，直到流被归约成一个值。
- 使用`reduce`可以让内部实现得以选择并行执行操作。但是，传递给`reduce`的Lambda不能更改状态（如实例变量），而且操作必须满足结合律才可以按任意顺序执行。

## 数值流

- `IntStream`、`DoubleStream`和`LongStream`分别将流中的元素特化为`int`、`double`和`long`，从而避免的暗含的装箱成本。数值流包含进行常用归约的方法，如求和`sum`，找到最大元素`max`等。
- 映射到数值流可以用`mapToInt`、`mapToDouble`和`mapToLong`，而使用`boxed`方法可以将数值流转回对象流（各个基本类型对应的包装类型）。
- `range`和`rangeClosed`是可以用于`IntStream`和`LongStream`的静态方法，作用是生成范围内的数，区别是`range`不包含结束值而`rangeClosed`包含。

## 构建流



