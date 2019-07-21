---
title: Java编程思想笔记0x11
date: 2019-07-18 17:06:13
categories: Java
tags:
  - Java
  - 笔记
---

# 枚举类型（一）

## 基本enum特性

- 调用`enum`的`values()`方法，可以遍历`enum`实例，返回一个`enum`实例的数组，并且数组中实例的顺序和声明时的顺序保持一致。
- 创建`enum`时，编译器会为你生成一个相关的类，这个类继承自`java.lang.Enum`。
- `ordinal()`方法返回一个`int`值，表示每个`enum`实例在声明中的次序，从`0`开始计算。
- 可以使用`==`来比较`enum`实例，编译器会自动提供`equals()`和`hashCode()`方法。
- `Enum`实现了`Comparable`接口，拥有`compareTo()`方法。
- `name()`方法返回`enum`实例声明时的名字，和`toString()`方法相同。
- `valueOf()`根据参数给出的实例名称返回相应的`enum`实例，如果不存在该名称的实例将会抛出异常。

> `enum`是java中的语法糖，实际反编译后是继承自`Enum`类。

## 向enum中添加新方法

- `enum`除了不能被继承，和普通的类基本相同。

```java
public enum Main {
    WEST("west"),
    NORTH("north"),
    EAST("east"),
    SOUTH("south");
    private String desc;
    Main(String desc) {
        this.desc = desc;
    }
    public String getDesc() {
        return desc;
    }
    public static void main(String[] args) {
        for (Main m: Main.values()) {
            System.out.println(m + " " + m.getDesc());
        }
    }
}
/* Output:
WEST west
NORTH north
EAST east
SOUTH south
*/
```

- 如果希望为`enum`添加方法，必须先定义`enum`实例，并且在实例序列最后添加一个分号。否则在编译时会得到错误消息。
- 可以为`enum`添加构造方法，注意声明`enum`实例时就是在使用`enum`的构造方法。此外，`enum`的构造方法无所谓是否`private`，因为就算不声明`private`其构造方法也只能在`enum`内部使用。
- 可以覆盖原来的`toString()`方法，以提供定制化效果。

> `enum`的构造方法不能被反射调用，当反射调用构造方法时会检查对应类是否为枚举类。

## switch语句中的enum

- 在`case`语句中不需要`enum`类来修饰`enum`实例。

## values()方法

```java
// 反编译enum代码
final class Apple extends java.lang.Enum<Apple> {
  public static final Apple A;
  public static final Apple B;
  public static final Apple C;
  public static Apple[] values();
  public static Apple valueOf(java.lang.String);
  static {};
}
```



- `values()`方法和`valueOf(String)`方法是编译器向`enum`中加入的静态方法。
- 编译器将`enum`标记为`final`类，因此无法继承`enum`。

## 随机选取

```java
public class Enums {
    private static Random rand = new Random(42);
    public static <T extends Enum<T>> T random(Class<T> ec) {
        return random(ec.getEnumConstants());
    }
    public static <T> T random(T[] values) {
        return values[rand.nextInt(values.length)];
    }
}
```

- `<T extends Enum<T>>`限制泛型必须为`enum`，使用`Class<T>`可以获取枚举的实例数组。

## 使用接口组织枚举

```java
interface Food {
    enum Appetizer implements Food {
        SALAD, SOUP, SPRING_ROLLS
    }
    enum MainCourse implements Food {
        LASAGNE, BURRITO, PAD_THAI, LENTILS
    }
    enum Dessert implements Food {
        TIRAMISU, GELATO, BLACK_FOREST_CAKE
    }
    enum Coffee implements Food {
        BLACK_COFFEE, DECAF_COFFEE, ESPRESSO
    }
}

public class Main {
    public static void main(String[] args) throws Exception {
        Food f = Food.Appetizer.SALAD;
        f = Food.Coffee.BLACK_COFFEE;
        f = Food.Dessert.BLACK_FOREST_CAKE;
    }
}
```

- 在一个接口的内部，创建实现该接口的枚举，以此将元素进行分组，可以达到将枚举元素分类组织的目的。

- 