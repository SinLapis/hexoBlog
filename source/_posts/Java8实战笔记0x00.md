---
title: Java8实战笔记0x00
date: 2019-07-29 11:31:24
categories: Java
tags:
  - Java
  - 笔记
---

# 通过行为参数化传递代码

## 行为参数化

- 谓词：根据对象的某些属性来返回`boolean`值的函数。

## 代码进化节选

### 使用接口实现策略设计模式

```java
// 顶层接口
public interface ApplePredicate {
    boolean test(Apple apple);
}
// 各种具体实现，可以不断增加
public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}
public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return "green".equals(apple.getColor());
    }
}
// 使用部分，无需变动
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory) {
        if (p.test(apple)) result.add(apple);
    }
    return result;
}
```

### 使用匿名内部类

- 在上面代码中，所定义的类可能仅实例化一次， 而使用匿名内部类可以弥补这点。

```java
// 仍然需要上面代码的顶层接口，但无需实际定义类
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
    public boolean test(Apple a) {
        return "red".equals(a.getColor());
    }
});
```

### 使用Lambda表达式

```java
List<Apple> result = filterApples(inventory, (Apple apple) -> "red".equals(apple.getColor()));
```

### 使用泛型

```java
public interface Predicate<T> {
    boolean test(T t);
}
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for (T e: list) {
        if (p.test(e)) result.add(e);
    }
    return result;
}
//...
List<Apple> redApples = filter(inventory, (Apple apple) -> "red".equals(apple.getColor()));
List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
```

# Lambda表达式

## 使用Lambda表达式

- 函数式接口：是指只定义一个抽象方法的接口。例如Java API中的`Comparetor`、`Runnable`、`Callable`等。
- `@FunctionalInterface`：用于标注接口为函数式接口，如果该接口没有定义抽象方法或者包含多个抽象方法，编译时会出现错误。其作用类似于`@Override`。

## 使用函数式接口

- `java.util.function.Predicate<T>`提供一个`test()`方法，接受一个泛型`T`对象，返回一个`boolean`。
- `java.util.function.Consumer<T>`提供一个`accept()`方法，接受一个泛型`T`对象，返回为`void`。
- `java.util.function.Function<T, R>`提供一个`apply()`方法，接受一个泛型`T`对象，返回一个泛型`R`对象。

### 原始类型特化

- 虽然Java可以进行自动装箱和拆箱，但是这一操作还是有性能代价的。为避免自动装箱和拆箱，Java为其提供的函数式接口提供了原始类型特有版本，例如功能类似`Predicate<T>`的`IntPredicate`，其参数为`int`而不是`Integer`可以避免装箱。

### 异常处理

- 任何函数式接口都不允许抛出受检异常。如果希望Lambda表达式可以抛出异常，可以定义一个自己的函数式接口，并声明受检异常，或者把Lambda主体用`try/catch`块包裹。下面代码为示例。

```java
@FunctionalInterface
public interface BufferredReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
//...
BufferedReaderProcessor p = (BufferedReader br) -> br.readLine();
```

```java
Function<BufferedReader, String> f = (BufferedReader b) -> {
    try {
        return b.readLine();
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
};
```

## 类型推断

- 编译器会自动推断Lambda表达式的参数类型，而无需显式声明。

```java
// 没有类型推断
Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
// 有类型推断
Comparator<Apple> c = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

## 使用局部变量

- Lambda表达式允许使用自由变量（在外层作用域中定义的变量），类似于匿名类。
- Lambda可以没有限制的捕获实例变量和静态变量，但是局部变量必须声明为`final`或者事实上是`final`。关于事实上是`final`的变量，见下面代码。不符合事实上是`final`的变量出现时，编译器将会发出错误信息。

```java
int n = 1337;
Runnable r = () -> System.out.println(n); 
// Error: java: 从lambda 表达式引用的本地变量必须是最终变量或实际上的最终变量
n = 31337; 
```

## 方法引用

- 方法引用可以被看作仅仅调用特定方法的Lambda的一种快捷写法。如果一个Lambda代表的只是直接调用这个方法，那最好还是用名称调用而不是尝试去描述过程。事实上，方法引用就是根据已有的方法实现来创建Lambda表达式。

```java
(Apple a) -> a.getWeight(); 
// 等价于
Apple::getWeight
```

### 分类

- 方法引用主要有三类：

  1. 指向静态方法的方法引用（例如`Integer`的`parseInt()`，写作`Integer::parseInt`）
  2. 指向任意类型实例方法的方法引用（例如`String`对象的`length()`，写作`String::length`）
  3. 指向现有对象的实例方法的方法引用（例如局部变量`x`，其拥有实例方法`getValue()`，则可写`x::getValue`）

  注意，第二种实例是Lambda本身的参数，而第三种实例来自外部。

### 构造函数引用

- 可以使用`ClassName::new`来对构造函数进行引用。注意选择正确的接口接收构造函数引用，见下面代码示例。

```java
class Apple {
    private int w;

    public Apple() {
        w = 10;
    }

    public Apple(int w) {
        this.w = w;
    }
    public static void main(String[] args) throws Exception {
        Supplier<Apple> c = Apple::new;
        Apple a1 = c.get();
        Function<Integer, Apple> c2 = Apple::new;
        Apple a2 = c2.get(20);
    }
}
```

`ClassName::new`会选择匹配函数式接口对应方法参数列表的构造函数，而无需指定参数列表。

## 复合Lambda表达式

- Java 8的一些函数式借口都有为方便而设计的方法，可以把多个简单的Lambda复合成复杂的表达式（例如使用`or`连接两个Lambda）。这些方法的实现使用了接口的默认方法。

### 比较器复合

以一个`Comparator`为例：

```java
Comparator<Apple> c = Comparator.comparing(Apple::getWeight);
```

#### 逆序

- 如果希望将结果逆序输出，可以使用默认方法`reversed()`将比较器结果逆序：

```java
inventory.sort(comparing(Apple::getWeight).reversed());
```

#### 比较器链

- 如果使用比较器得到两个应该处于相同位置的对象时，可以使用`thenComparing()`来进一步比较：

```java
inventory.sort(comparing(Apple::getWeight))
         .reversed()
         .thenComparing(Apple::getCountry));
```

### 谓词复合

- 谓词借口包括三个方法：`negate`、`and`和`or`，其优先级是从左往右。

```java
// 从现有的Predicate创建结果的非
Predicate<Apple> notRedApple = redApple.negate();
// 使用and和or创建复杂的Predicate对象
Predicate<Apple> p = redApple.and(a -> a.getWeight() > 150)
                             .or(a -> "green".equals(a.getColor()));
```

### 函数复合

- 可以使用`Function`接口将Lambda表达式复合起来。其中提供`andThen()`和`compose()`两个方法，它们都会返回一个`Function`实例。`andThen()`会先对输入应用调用该方法的`Function`，后对输入应用该方法的`Function`参数；`compose()`则是先对输入应用该方法的`Function`参数，后对输入应用调用该方法的`Function`

