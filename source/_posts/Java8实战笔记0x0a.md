---
title: Java8实战笔记0x0a
date: 2019-08-24 10:08:36
categories: Java
tags:
  - Java
  - 笔记
  - 函数式
---

# 函数式编程的技巧

## 无处不在的函数

### 高阶函数

- 函数满足以下任一要求即可被称为告诫函数：
  - 接受至少一个函数作为参数
  - 返回结果是一个函数

```java
Comparator<Apple> c = comparing(Apple::getWeight);
```

### 科里化

- 科里化是指一个能将具有`n`个参数的函数转化为使用`n`个参数中部分参数的函数，该函数的返回值是另一个函数，参数为转化的函数未使用的参数，例如`f(x, y) =  (g(x))(y)`

```java
// 摄氏度转华氏度
static DoubleUnaryOperator curriedConverter(double f, double b) {
    return (double x) -> x * f + b;
}

DoubleUnaryOperator convertCtoF = curriedConverter(9.0/5, 32);
double c = convertCotF(25);
```

- 一个函数使用所有参数仅有部分被传递时，通常称这个函数是部分应用的。

## 延迟计算

### 自定义延迟列表

```java
interface MyList<T> {
    T head();

    MyList<T> tail();

    default boolean isEmpty() {
        return true;
    }

    default public MyList<T> filter(Predicate<T> p) {
        return isEmpty() ? this :
                p.test(head()) ?
                        new LazyList<>(head(), () -> tail().filter(p)) :
                        tail().filter(p);
    }
}

class MyLinkedList<T> implements MyList<T> {
    private final T head;
    private final MyList<T> tail;

    public MyLinkedList(T head, MyList<T> tail) {
        this.head = head;
        this.tail = tail;
    }

    @Override
    public T head() {
        return head;
    }

    @Override
    public MyList<T> tail() {
        return tail;
    }

    @Override
    public boolean isEmpty() {
        return false;
    }


}

class Empty<T> implements MyList<T> {
    @Override
    public T head() {
        throw new UnsupportedOperationException();
    }

    @Override
    public MyList<T> tail() {
        throw new UnsupportedOperationException();
    }
}

class LazyList<T> implements MyList<T> {
    final T head;
    final Supplier<MyList<T>> tail;

    public LazyList(T head, Supplier<MyList<T>> tail) {
        this.head = head;
        this.tail = tail;
    }

    @Override
    public T head() {
        return head;
    }

    @Override
    public MyList<T> tail() {
        return tail.get();
    }

    @Override
    public boolean isEmpty() {
        return false;
    }

}

public class Main {
    public static void test(Function<String, List<String>> findPrices) {
        long start = System.nanoTime();
        System.out.println(findPrices.apply("Fallout New Vegas"));
        long duration = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Done in " + duration + " msecs");
    }

    public static MyList<Integer> primes(MyList<Integer> numbers) {
        return new LazyList<>(
                numbers.head(),
                () -> primes(
                        numbers.tail().filter(n -> n % numbers.head() != 0)
                )
        );
    }

    public static LazyList<Integer> from(int n) {
        return new LazyList<>(n, () -> from(n + 1));
    }
    static <T> void printAll(MyList<T> list) {
        while (!list.isEmpty()) {
            System.out.println(list.head());
            list = list.tail();
        }
    }
    public static void main(String[] args) throws Exception {
        printAll(primes(from(2)));
    }
}
```

~~没看懂~~

## 模式匹配

- Java没有原生支持模式配配匹配，只能用`if`进行模拟。