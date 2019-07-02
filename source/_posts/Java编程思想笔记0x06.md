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