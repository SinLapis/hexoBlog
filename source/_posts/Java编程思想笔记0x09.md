---
title: Java编程思想笔记0x09
date: 2019-07-07 09:42:32
categories: Java
tags:
  - Java
  - 笔记
---

# 泛型（四）

## 动态类型安全

- 因为可以向旧版本的Java代码中传递容器，而如果旧版本的代码没有使用泛型则需要动态类型检查。`java.uitl.Collections`中有一系列方法可以完成这项工作：`checkedCollection()`、`checkedList()`、`checkedMap()`、`checkedSet()`、`checkedSortedMap()`、`checkedSortedSet()`。其参数为希望获得检查的容器和强制要求的类型。受检查的容器在插入不正确的类型时会抛出`ClassCastException`。例如：

  ```List<Cat> cats = Collections.checkedList(new ArrayList<>(), Cat.class);```

  此时插入`Cat`对象是正确的，而插入`Dog`对象则会出现异常。

## 异常

- 由于擦除的原因，将泛型应用于异常是非常受限的。`catch`语句不能捕获泛型类型异常，因为在编译期和运行时都必须知道异常的确切类型。泛型类也不能直接或间接继承`Throwable`。
- 在方法的`throw`子句可以使用类型参数，使得随检查异常的类型发生变化而变化的代码

## 混型

- 混型是指回合多个类的能力，以产生一个可以表示混型中所有类型的类。
- 混型的价值之一是可以将特性和行为一致地应用于多个类桑，如果在混型类中修改，那么这些修改将会应用于混型所应用的所有类型之上。混型有一点面向切面编程的意思。

### 与接口混合

- 一种常见的混型方法就是使用接口产生混型效果，即使用代理，每个混入类型都有一个相应的域，调用方法时要转发给对应的域。

### 使用装饰器模式

- 装饰器是通过使用组合和形式化结构（可装饰物/装饰器层器结构）类实现的，而混型是基于继承的。因此可以将基于参数化类型的混型当做一种泛型装饰器机制，这种机制不需要装饰器设计模式的继承结构。（*即声明一个混型，其泛型类型参数为混型基于的所有类型，以达到混型的目的*）

### 与动态代理混合

- 可以使用动态代理实现更为接近混合模型的机制。通过使用动态代理，所产生的类的动态类型将会是已经混入的组合类型。
- 由于动态代理的限制，每个被混入的类都必须是某个接口的实现。
- 在调用混入类型的方法之前，需对混型对象进行向下转型到合适的类型。

## 潜在类型机制

- 潜在类型机制，又称鸭子类型机制，具有这种机制的语言只要求实现某个方法的自己，而不是某个特定类型或者接口，从而放松了限制。潜在类型机制可以横跨类型继承结构，调用某个不是公共接口的方法。
- 在Java中是没有原生实现的潜在类型机制，会被强制要求继承某个类或者实现某个接口。

## 对缺乏潜在类型机制的补救

### 反射

```java
class CommunicateReflectively {
    public static void perform(Object speaker) {
        Class<?> spkr = speaker.getClass();
        try {
            Method speak = spkr.getMethod("speak");
            speak.invoke(speaker);
        } catch(NoSuchMethodException e) {
            System.out.println(speaker + "cannot speak");
        }
    }
}
```

上面的代码使用了反射获取了指定名称的方法，但是类型检查转移到了运行时。

### 将一个方法应用于序列

```java
public class Apply {
    public static <T, S extends Iterable<? extends T>> 
        void apply(S seq, Method f, Object... args) {
        try {
            for (T t: seq) {
                f.invoke(t, args);
            }
        } catch(Exception e) {
            throw new RuntimException(e);
        }
    }
}
```

- 上面代码的目的是实现对任意的序列`S<T>`调用某种方法，然而恰好序列都是实现了`Iterable`接口的。但是这种方法并不适用于没有实现内建接口或者继承内建类的类型。

### 用适配器模式仿真潜在类型机制

- 可以使用适配器模式来适配已有的接口，来产生需要的接口。

# 数组

## 数组的特点

- 效率：数组是一种效率最高的存储和随机访问对象引用序列的方式，代价是数组对象的大小被固定，并且在其生命周期中不能改变。
- 类型：相比缺少泛型的容器，数组可以持有某种类型的对象，可以通过编译期检查来防止不当的操作。
- 持有基本类型：数组可以持有基本类型，但是容器不能直接持有基本类型。

