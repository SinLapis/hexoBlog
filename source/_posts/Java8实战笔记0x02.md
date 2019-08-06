---
title: Java8实战笔记0x02
date: 2019-08-05 20:40:09
categories: Java
tags:
  - Java
  - 笔记
---

# 用流收集数据

## 归约和汇总

### 查找流中的最大值和最小值

- `Collectors.maxBy`和`Collectors.minBy`可以计算流中的最大或最小值。这两个收集器接收一个`Comparator`参数来比较流中的元素。

```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish = menu.stream()
    .collect(Collectors.maxBy(DishCaloriesComparator));
```

### 汇总

- `Collectors`类专门为汇总提供了一个工厂方法`Collectors.summingInt`。它可接受一个把对象映射为求和所需`int`的函数，并返回一个收集器；该收集器在传递普通的`collect`方法后即执行所需要的汇总操作。类似的还有`Collectors.summingLong`和`Collectors.summingDouble`。

```java
int totalCalories = menu.stream().collect(summingInt(Dish:getCalories));
```

- `Collectors.averagingInt`，以及对应的`Collectors.averagingLong`和`Collectors.averagingDouble`可以计算数值的平均数。

```java
double avgCalories = menu.stream().collect(averagingInt(Dish::getCalories));
```

- `summarizingInt`工厂方法返回的收集器可以一次操作就得到流中元素的个数、所选值的总和、平均值、最大值、最小值。

```java
IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
```

### 连接字符串

- `joining`工厂方法返回的收集器会把对流中每一个对象应用`toString`方法得到的所有字符串连接成一个字符串。

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining())
```

- `joining`在内部使用了`StringBuilder`来把生成的字符串逐个追加起来。此外，如果对象有`toString()`方法，那么可以省略`map`映射。

```java
String shortMenu = menu.stream().collect(joining());
```

- `joining()`还接受一个字符串作为分界符。

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
```

### 广义的归约汇总

- `reduce`工厂方法则是所有归约情况的一般化，它需要三个参数：初始值、转换函数、累积函数（将两个项目累积成一个同类型的值）。

```java
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, Integer::sum));
```

## 分组

- 可以使用`Collectors.groupingBy`工厂方法返回的收集器来实现元素分组。

```java
Map<Dish.Type, List<Dish>> dishesByType = menu.stream(groupingBy(Dish::getType));
```

- 在上面代码中，`groupingBy`方法接受一个分类函数，用于提取元素的类别从而对元素进行分组。分组的操作结果是一个`Map`，分组函数返回的值作为映射的键，流中所有具有这个分类值的项目的列表作为对应的映射值。如果没有现成的获取元素类别的函数，可以传入Lambda。

```java
public enum ColaricLevel { DIET, NORMAL, FAT }

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(
	groupingBy(dish -> {
        if (dish.getCalories() <= 400) return CaloricLevel.DIET;
        else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
        else return CaloricLevel.FAT;
    })
);
```

### 多极分组

- `Collectors.groupingBy`有双参数版本，第二个参数为`collector`类型。进行二级分组时，可以把一个内层`groupingBy`传递给外层`groupingBy`，并定义一个为流中项目分类的二级标准。

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = menu.stream().collect(
	groupingBy(Dish::getType, groupingBy(dish -> {
        if (dish.getCalories() <= 400) return CaloricLevel.DIET;
        else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
        else return CaloricLevel.FAT;
    }))
);
```

### 按子组收集数据

- `groupingBy`的第二个参数也可以是其它`collect`。

```java
// 分组求和
Map<Dish.Type, Long> typesCount = menu.stream().collect(groupingBy(Dish::getType, counting()));

// 找出分组最大值
Map<Dish.Type, Optional<Dish>> mostCaloricByType = menu.stream().collect(groupingBy(
	Dish::getType,
    maxBy(comparingInt(Dish::getCalories))
));
```

- `Collectors.collectingAndThen`可以把收集器返回的结果转换为另一种类型。

```java
Map<Dish.Type, Dish> mostCaloricByType = menu.stream().collect(groupingBy(
	Dish::getType,
    collectingAndThen(maxBy(comparingInt(Dish::getCalories)), Optional::get)
));
```

