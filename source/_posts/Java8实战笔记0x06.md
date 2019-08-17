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
- 破坏哲学：Java一直试图避免让程序员意识到指针的存在，但`null`指针例外。
- 类型缺失：`null`不属于任何类型，这意味着它可以被赋值给任何变量，而当这个变量传递给系统的其它部分时，将无法确定`null`变量最初是什么类型。