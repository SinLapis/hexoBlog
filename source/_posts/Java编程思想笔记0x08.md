---
title: Java编程思想笔记0x08
date: 2019-07-05 18:57:32
categories: Java
tags:
  - Java
  - 笔记
---

# 泛型（三）

## 问题

### 任何基本类型都不能作为类型参数

- Java泛型中不能使用基本类型用作类型参数，取而代之可以使用基本类型的包装器以及自动包装机制。

### 实现参数化接口

- 一个类不能实现同一个泛型接口的两种变体，由于擦除的原因，这两个变体会成为相同的接口。

### 转型和警告

- 使用带有泛型类型参数的转型或`instanceof`不会有任何效果。因为类型参数`T`会被擦除到第一个边界，默认为`Object`，使用转型或`instanceof`也只使用了`Object`。
- 在必须进行转型到某种泛型类时，需要使用泛型类对象转型，例如`List.class.cast(x)`，但是无法转换到具体类型，即不能使用`List<RealClass>.class.cast(x)`。

### 重载

- 由于擦除的原因，相同泛型类、不同类型参数的重载方法将产生相同的类型签名。

### 基类劫持接口

- 当父类实现泛型接口时，实现了接口中的某个带有相应泛型参数的方法，此时子类将不能修改该方法的参数类型。

## 自限定的类型

### 古怪的循环泛型

- 诸如`class Sub extends Basic<Sub>`的定义就是古怪的循环泛型，子类类型出现在了基类中。
- 其本质是基类用导出类替代其参数，意味着泛型基类变成了一种其所有导出类的公共功能的模板，但是这些功能对于其所有的参数和返回值，将使用导出类型。

### 自限定

- 下面代码中`SelfBounded`即自限定的泛型基类。

```java
class SelfBounded<T extends SelfBounded<T>> {
    T element;
    SelfBounded<T> set(T arg) {
        element = arg;
        return this;
    }
    T get() {
        return element;
    }
}
```

继承`SelfBounded`必须类似与`class A extends SelfBounded<A>`来定义子类，这保证了类型参数必定与正在被定义的类相同。

同样的，自限定还可以用于泛型方法，防止该方法用于自限定参数之外的任何事物上：

```java
public class SelfBoundingMethods {
    static <T extends SelfBoundingMethods<T>> T f (T arg) {
        return arg.set(arg).get();
    }
    public static void main(String[] args) {
        A a = f(new A());
    }
}
```

### 参数协变

- 自限定类型的意义在于能够产生协变参数类型，即方法参数类型会随着子类变化而变化，不会出现重载。

*但实际上如果仅为了实现参数协变，自限定并不是必要的，使用过循环泛型就能解决，例如：*

```java
public class Test {
    public static void main(String[] args) {
        A a = new A();
        System.out.println(a.set(a));
        System.out.println(a);
        System.out.println(a.get());
        B b = new B();
        System.out.println(b.set(b));
        System.out.println(b);
        System.out.println(b.get());
    }
}
class SelfBounded<T extends SelfBounded<T>> {
    T element;
    SelfBounded<T> set(T arg) {
        element = arg;
        return this;
    }
    T get() {
        return element;
    }
}
class Cyclic<T> {
    T element;
    Cyclic<T> set(T arg) {
        element = arg;
        return this;
    }
    T get() {
        return element;
    }
}
class A extends SelfBounded<A> {}
class B extends Cyclic<B> {}
/* Output:
A@2c13da15
A@2c13da15
A@2c13da15
B@9e89d68
B@9e89d68
B@9e89d68
*/
```

*上面代码中，`SelfBounded`是自限定的，而`Cyclic`仅是普通的泛型类，其子类`B`使用了循环泛型就实现了参数协变。两者唯一的不同仅在于自限定中类型参数必须是自限定的，而循环泛型并无此限制*

