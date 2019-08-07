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

### 多级分组

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

## 分区

- 分区是分组的特殊情况：由一个谓词作为分类函数，它成为分区函数。分区函数返回一个布尔值，这意味着得到的分组`Map`的键类型是`Boolean`，于是它最多可以分为两组，`true`一组`false`一组。类似`groupingBy`可以进行多级分区。

```java
Map<Boolean, List<Dish>> partitionedMenu = menu.stream().collect(partitioningBy(Dish::isVegetarian));
```

## 收集器接口

```java
// Collector接口定义
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Characteristics> characteristics();
}
```

### 接口解析

#### 泛型

- `T`是流中要收集的项目的泛型；`A`是累加器的类型，累加器是在收集过程中用于累积部分结果的对象；`R`是收集操作得到的对象（一般情况下是集合，但也有可能不是）。

#### 方法

- `supplier`方法：用于创建一个空的累加器实例，供数据收集过程使用

```java
public Supplier<List<T>> suplier() {
    return () -> new ArrayList<T>;
}
// 或者传递构造函数引用
public Supplier<List<T>> suplier() {
    return ArrayList::new;
}
```

- `accumulator`方法：返回执行归约操作的函数。当遍历到流中第n个元素时，这个函数执行时会有两个参数，保存归约结果的累加器（已收集了流中的n - 1个项目），以及第n个元素本身。该函数将返回`void`，因为是累加器的原位更新，即函数的执行改变了它的内部状态以体现遍历的元素的效果。

```java
public BiConsumer<List<T>, T> accumulator() {
    return (list, item) -> list.add(item);
}
// 或者传递构造函数引用
public BiConsumer<List<T>, T> accumulator() {
    return List::add;
}
```

- `finisher`方法：在遍历完流后，`finisher`方法必须返回累积过程的最后要调用的一个函数，以便将累加器对象转换为整个集合操作的最终结果。如果不需要进行转换，可以返回`Function.identity()`，该函数直接返回对象本身，即`return t -> t;`。

```java
public Function<List<T>, List<T>> finisher() {
    return Function.identity();
}
```

- `combiner`方法：返回一个供归约操作使用的函数，它定义了对流的各个子部分进行并行处理时，各个子部分归约所得到的累加器要如何合并。

```java
public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> {
        list1.addAll(list2);
        return list1;
    }
}
```

- 并行归约流程

1. 原始流会以递归方式拆分为子流，直到定义流是否需要进一步拆分的一个条件为非。
2. 对所有的子流并行处理，即对每个子流应用归约算法。
3. 使用收集器`combiner`方法返回的函数，将所有的部分结果两两合并。

- `characteristics`方法：返回一个不可变的`Characteristics`集合，它定义了收集器的行为，尤其是关于流是否可以并行归约，以及可以使用哪些优化的提示。`Characteristics`是一个包含三个项目的枚举：
  1. `UNORDERED`：归约结果不受流中项目的遍历和累积顺序的影响
  2. `CONCURRENT`：`accumulator`函数可以从多个线程同时调用，且该收集器可以并行归约流。如果收集器没有标为`UNORDERED`，那它仅在用于无序数据源才可以并行归约。
  3. `IDENTITY_FINISH`：这表明完成器方法返回的函数是一个恒等函数，可以跳过。这种情况下，类累加器对象将会直接用作归约过程的最终结果。这也就意味着，将累加器`A`不加检查地转换为结果`R`是安全的。

### 进行自定义收集而不实现接口

- `collector`的一个重载版本可以接受三个参数，`supplier`、`accumulator`、`combiner`。

```java
List<Dish> dishes = menuStream.collect(
	ArrayList::new,
    List::add,
    List::addAll
);
```



