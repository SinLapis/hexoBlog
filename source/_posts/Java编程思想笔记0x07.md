---
title: Java编程思想笔记0x07
date: 2019-07-04 19:22:09
categories: Java
tags:
  - Java
  - 笔记
---

# 泛型（二）

## 边界

- 泛型边界不但可以强制规定泛型可以应用的类型，还允许泛型按照其边界类型调用方法。

## 通配符

> 逆变与协变用来描述类型转换后的继承关系，其定义为：如果A、B表示类型，f(⋅)表示类型转换，≤表示继承关系（比如，A≤B表示A是由B派生出来的子类）
> f(⋅)是逆变（contravariant）的，当A≤B时有f(B)≤f(A)成立；
> f(⋅)是协变（covariant）的，当A≤B时有f(A)≤f(B)成立；
> f(⋅)是不变（invariant）的，当A≤B时上述两个式子均不成立，即f(A)与f(B)相互之间没有继承关系。

- 在Java中，数组是协变的

```java
class Fruit {}
class Apple extends Fruit {}
class Jonathan extends Apple {}
class Orange extends Fruit {}

public class Test {
    public static void main(String[] args) {
        Fruit[] fruits = new Apple[10];
        fruits[0] = new Apple();
        fruits[1] = new Jonathan();
        try {
            fruits[0] = new Fruit(); // Runtime Error
        } catch (Exception e) {
            System.out.println(e);
        }
        try {
            fruits[1] = new Orange(); // Runtime Error
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
```

上述代码可以通过编译，但是在运行时会报错。

*由于早期Java不支持泛型，对数组的通用操作都是使用`Object[]`实现的，因此上述代码可以通过编译。但是为了避免因为声明类型和实际类型不一致而引发的问题，Java把类型检查放在了运行时*

然而使用泛型时，由于擦除的存在，所有检查都必须在编译期完成，因此诸如`List<Fruit> flist = new ArrayList<Apple>;`的代码是无法通过编译的。与数组不同，泛型没有内建的协变类型。

但是这一限制可以使用通配符解除，例如`List<? extends Fruit> flist = new ArrayList<Apple>;`，这不意味着`flist`可以持有任意`Fruit`及其子类的对象，仍然需要指明其持有类型，并向上转型。

此时问题又出现了，向上转型后将无法传入任何类型对象给`flist`，因为编译器不知道`<? extends Fruit>`指向哪个类型，例如它可以指向`Orange`，那么向其中放入`Apple`、`Fruit`、`Object`对象都是非法的。不过此时从`flist`中取出`Apple`对象是允许的（前提是容器中有对象）。

### 逆变

- 使用超类型通配符可以允许向持有某种类型的容器中写入其子类，即指定泛型`<? super ClassName>`，甚至可以使用类型参数`<? super T>`（但不能针对类型参数给出一个超类型边界`<T super ClassName>`）。

```java
public class SuperType {
    static void writeTo(List<? super Apple> apples) {
        apples.add(new Apple());
        apples.add(new Jonathan());
    }
}
```

下面代码中`writeExact()`无法将`Apple`对象放入`List<Fruit>`中，即使是允许的。而使用了超类型通配符的`writeWithWildcard()`则以把`Apple`对象放入`List<Fruit>`。

```java
class GenericWriting {
    static <T> void writeExact(List<T> list, T item) {
        list.add(item);
    }
    static List<Apple> apples = new ArrayList<>();
    static List<Fruit> fruits = new ArrayList<>();
    static void f1() {
        writeExact(apples, new Apple());
        //writeExact(fruits, new Apple()); // Error
    }
    static <T> void writeWithWildcard(List<? super T> list, T item) {
        list.add(item);
    }
    static void f2() {
        writeWithWildcard(apples, new Apple());
        writeWithWildcard(fruits, new Apple());
    }
}
```

相对应的，读取的代码可以使用子类型通配符，使读取对象时实现向上转型。

```java
class GenericReading {
    static <T> T readExact(List<T> list) {
        return list.get(0);
    }
    static List<Apple> apples = Arrays.asList(new Apple());
    static List<Fruit> fruits = Arrays.asList(new Fruit());
    static void f1() {
        Apple a = readExact(apples);
        Fruit f = readExact(fruits);
        f = readExact(apples);
    }
    
    static class Reader<T> {
        T readExact(List<T> list) {
            return list.get(0);
        }
    }
    static void f2() {
        Reader<Fruit> fruitReader = new Reader<>();
        //Fruit a = fruitReader.readExact(apples); // Error
    }
    static class CovariantReader<T> {
        T readCovariant(List<? extends T> list) {
            return list.get(0);
        }
    }
    static void f3() {
        CovariantReader<Fruit> fruitCovariantReader = new CovariantReader<>();
        Fruit f = fruitCovariantReader.readCovariant(fruits);
        Fruit a = fruitCovariantReader.readCovariant(apples);
    }
}
```

在上面的代码中，静态方法`readExact()`由于类型参数由`list`决定，所以正确返回了`Apple`对象并向上转型。如果只是读取，可以不使用泛型。而在`f2()`中，由于创建泛型类时先指定了类型参数为`Fruit`，因此`Reader#readExact()`不能接受`List<Apple>`参数。此时使用子类通配符即可解决问题。

### 无界通配符

- 使用无界通配符表示，当前不知道（或者不需要知道）具体类型，但是不影响对其进行操作。例如可以从`List<?>`中取值出来（但是会丢失类型信息），不可以向其中写入。

### 捕获转换

- 如果向一个使用`<?>`的方法传递原生类型，对于编译器来说，可能会推断出实际的类型参数，使得这个方法可以回转并调用另一个使用这个确切类型的方法，即捕获转换。

```java
public class Main {
    static <T> T f1(List<T> list) {
        T t = list.get(0);
        return t;
    }

    static void f2(List<?> list) {
        f1(list);
    }

    public static void main(String[] args) {
        List list = new ArrayList<>();
        list.add(0);
        f1(list);
        f2(list);
    }
}
```

上面代码中，`main()`方法中调用`f1(list)`是有警告的，而经过`f2()`的捕获转换后警告消失了。

## 问题



