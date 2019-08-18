---
title: Java8实战笔记0x06
date: 2019-08-17 10:48:10
categories: Java
tags:
  - Java
  - 笔记
---

# 用Optional取代null

## 如何为缺失值建模

```java
class Person {
    private Car car;
    public Car getCar() {
        return car;
    }
}

class Car {
    private Insurance insurance;

    public Insurance getInsurance() {
        return insurance;
    }
}

class Insurance {
    private String name;

    public String getName() {
        return name;
    }
}

public class Main {
    public static String getCarInsuranceName(Person person) {
        return person.getCar().getInsurance().getName();
    }
}
```

- 一般情况下，如果`Person`没有`Car`，那么`getCar()`会设置为返回`null`，表示该值缺失。因此需要判断返回值是否为`null`来防止出现`NullPointerException`

### 采用防御式检查减少NullPointerException

- 深层质疑：会增加代码缩进层数，不具备扩展性，代码维护困难。

```java
public static String getCarInsuranceName(Person person) {
        if (person != null) {
            Car car = person.getCar();
            if (car != null) {
                Insurance insurance = car.getInsurance();
                if (insurance != null) {
                    return insurance.getName();
                }
            }
        }
        return "Unknown";
    }
```

- 过多的退出语句：退出点数量多，难以维护。

```java
public static String getCarInsuranceName(Person person) {
        if (person == null) {
            return "Unknown";
        }
        Car car = person.getCar();
        if (car == null) {
            return "Unknown";
        }
        Insurance insurance = car.getInsurance();
        if (insurance == null) {
            return "Unknown";
        }
        return insurance.getName();
    }
```

### null带来的问题

- 错误之源：`NullPointerException`是目前Java程序开发中最典型的异常。
- 代码膨胀：需要深度嵌套`null`检查，可读性极差。
- 毫无意义：`null`自身没有任何语义，是以一种错误的方式对缺失值建模。
- 违反哲学：Java一直试图避免让程序员意识到指针的存在，但`null`指针例外。
- 类型缺失：`null`不属于任何类型，这意味着它可以被赋值给任何变量，而当这个变量传递给系统的其它部分时，将无法确定`null`变量最初是什么类型。

### 其他语言中null的替代品

- Groovy：引入安全导航操作符，可以安全的访问可能为`null`的变量，避免抛出`NullPointerException`。

```groovy
def carInsuranceName = person?.car?.insurance?.name
```

- Haskell：`Maybe`类型，本质上是`optional`值的封装。
- Scala：`Option[T]`，要显式调用`avaliable`操作检查该变量是否有值，其实是变相的`null`检查。

## Optional类简介

- 变量存在时，`Optional`类型只是对对象的简单封装。变量不存在时，缺失的值会被建模成一个“空”的`Optional`对象，由方法`Optional.empty()`返回。`Optional.empty()`方法是一个静态工厂方法，它返回`Optional`类的特定单一实例。

```java
class Person {
    private Car car;
    
    public Optional<Car> getCar() {
        return Optional.ofNullable(car);
    }
}

class Car {
    private Insurance insurance;

    public Optional<Insurance> getInsurance() {
        return Optional.ofNullable(insurance);
    }
}

class Insurance {
    private String name;

    public String getName() {
        return name;
    }
}
```

- <del>`Optional`和`null`的重要语义区别是，`Optional`类清楚地表明允许发生变量缺失，`null`则不允许。</del>在代码中，如果变量不能为`null`，那么也不需要为其添加`null`检查，因为`null`的检查只会掩盖问题，并未真正修复问题。
- *书中对应上面代码部分在声明成员变量时形如`private Optional<Car> car`，但此时IDEA报出警告`'Optional' used as field or parameter type`，参考[StackOverFlow](<https://stackoverflow.com/questions/31922866/why-should-java-8s-optional-not-be-used-in-arguments>)，`Optional`作为成员变量或者函数参数类型会导致不必要的包装和额外的解包逻辑，且无法序列化，因此只需在返回值处使用`Optional`。*

## 应用Optional的几种模式

### 创建Optional对象

- 声明一个空的`Optional`

```java
Optional<Car> optCar = Optional.empty();
```

- 依据一个非空值创建Optional

```java
Optional<Car> optCar = Optional.of(car);
```

- 可接受`null`的`Optional`，如果传入`null`则获得空对象

```java
Optional<Car> optCar = Optional.ofNullable(car);
```

### 使用map从Optional对象中提取和转换值

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

- 如果`Optional`包含一个值，那函数就将该值作为参数传递给`map`，对该值进行转换。如果`Optional`为空，就什么也不做。

### 使用flatMap链接Optional对象

```java
Optional<Person> optPerson = Optional.of(person);
//！Optional<String> name = optPerson.map(Person::getCar)
//！    .map(Car::getInsurance)
//！    .map(Insurance::getName);
```

- 上面代码注释处无法通过编译。第一个`map`返回值类型是`Optional<Optional<Car>>`，而不是需要的`Optional<Car>`。
- 解决上述问题应该使用`flatMap`，`flatMap`方法接受一个函数作为参数，这个函数的返回值是另一个流。这个方法会应用到流的每一个元素，最终形成一个新的流。

```java
// 与书中略有不同，详情见上
public static String getCarInsuranceName(Person person) {
        return Optional.ofNullable(person)
                .flatMap(Person::getCar)
                .flatMap(Car::getInsurance)
                .map(Insurance::getName)
                .orElse("Unknown");
    }
```

### 默认行为及解引用Optional对象

- `get()`：如果变量存在则返回封装的变量值，否则抛出`NoSuchElementException`异常。不推荐使用。
- `orElse(T other)`：如果变量存在则返回，否则返回传入的默认值。
- `orElseGet(Supplier<? extends X> other)`：是`orElse()`的延迟调用版。传入的`Supplier`只有在变量不存在时调用。如果创建对象是耗时操作应该使用该方法。
- `orElseThrow(Supplier<? extends x> exceptionSupplier)`：类似于`get()`，不过抛出的错误由传入的`Supplier`创建。
- `ifPresent(Consumer<? extends T> consumer)`：在变量存在时执行传入的`Consumer`，否则就不进行任何操作。

### 两个Optional对象组合

```java
public Optional<Insurance> safeFind(Person person, Car car) {
    return Optional.ofNullable(person)
            .flatMap(
                   p -> Optional.ofNullable(car).map(c -> find(p, c))
            );
}
```

- 首先，`c -> find(p, c)`返回`Insurance`，而流的类型为`Optional`，因此不需要`flatMap`。之后`p -> Optional.ofNullable(car).map(c -> find(p, c))`得到的就是`Optional<Insurance>`，`Optional.ofNullable(person)`的流类型是`Optional<Person>`，因此需要`flatMap`。

### 使用filter剔除特定的值

- `filter`方法接受一个谓词作为参数。如果`Optional`对象的值存在，并且它符合谓词条件，`filter`方法就返回其值；否则它就返回一个空的`Optional`对象。

```java
public String getCarInsuranceName(Optional<Person> person, int minAge) {
    return person.filter(p -> p.getAge() >= minAge)
        .flatMap(Person::getCar)
        .flatMap(Car::getInsurance)
        .map(Insurance::getName)
        .orElse("Unknown");
}
```

## 使用Optional的实战示例

### 用Optional封装可能为null的值

```java
// 原始代码
Object value = map.get("key");
// if value != null ...

// 使用Optional
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

### 异常与Optional对比

```java
// 将String转为Integer
public static Optional<Integer> stringToInt(String s) {
    try {
        return Optional.of(Integer.parseInt(s));
    } catch (NumberFormatExcption e) {
        return Optional.empty();
    }
}

// Java 9中添加了Stream.ofNullable，如果对象为null，
public OptionalInt stringToInt(String s) {
    return Stream.ofNullable(s)
        .filter(str -> str.matches("\\d+"))
        .mapToInt(Integer::parseInt)
        .findAny();
}
```

