---
title: Java编程思想笔记0x06
date: 2019-07-02 20:33:00
categories: Java
tags:
  - Java
  - 笔记
---

# 泛型

## 简单泛型

- 让一个类能够持有多种类型的对象，可以使用泛型实现。

  ```java
  class A<T> {
      T i;
      A(T ii) {
          i = ii;
      }
  }
  
  public class Test {
      public static void main(String[] args) {
          A<Integer> a = new A<>(1);
          int i = a.i;
          // a.i = "str test"; // 错误
          A<String> b = new A<>("test");
          String s = b.i;
      }
  }
  ```

  

### 示例：元组

- 元组是指将一组对象直接打包存储于其中的一个单一对象，这个容器对象允许读取其中的元素，但是不允许向其中存放新的对象。

  ```java
  public class TwoTuple<A,B> {
      public final A first;
      public final B second;
      public TwoTuple(A a, B b) {
          first = a;
          second = b;
      }
      public String toString() {
          return "(" + first + ", " + second + ")";
      }
  }
  ```

  当希望某个方法返回两个及以上的参数是可以使用元组来实现，创建时填入类型即可。

## 泛型接口

- 泛型可以应用于接口，例如生成器：

  ```java
  public interface Generator<T> {
      T next();
  }
  ```

## 泛型方法

- 泛型同样可以在类中包含参数化方法，并且与类是否泛型无关。

- 如果使用泛型方法可以取代整个类泛型化，那么就应该只使用泛型方法

- 静态方法无法访问类成员中的泛型变量。如果希望静态方法拥有泛型能力，那么就需要让静态方法称为泛型方法。

- 定义泛型方法，只需将泛型参数列表置于返回值之前：

  ```java
  public <T> void f(T x){}
  ```

- 调用泛型方法时，不需要指明参数类型，编译器会自动判断类型，即类型参数推断。

> 在Java 8以前，泛型方法的结果传递给另外一个方法时编译器不会进行推断，而Java 8 中编译器能够根据调用的方法和相应的声明来确定需要的参数类型。

