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

## 匿名内部类

- 泛型可以用于匿名内部类，同样以生成器为例：

```java
public interface Generator<T> {
    T next();
}

public static Generator<String> generator() {
    return new Generator<String> {
        public String next() {
            return "test";
        }
    }
}
```

## 擦除

- Java的泛型是使用擦除来实现的，这意味着在泛型代码内部，无法获得任何有关泛型参数类型的信息，即实际上，`List<Integer>`和`List<String>`在运行时是相同的类型`List`。
- 泛型类型参数将擦除到它的第一个边界，实际上编译器会把类型参数替换为它的擦除。
- 当希望代码能够跨多个类工作时，使用泛型才有所帮助。例如某个类有一个返回`T`的方法，那么泛型可以帮助该方法返回正确的类型。

## 弥补擦除

- 在需要类型信息而其已经被擦除时，必须显式地传递类型的Class对象。例如，`x instanceof T`是错误的表达，需要x先用`Class<T> ct`对象保存类型信息，再比较类型信息`ct.isInstnce(x)`。

### 创建泛型类型对象

- 传递一个工厂对象到构造器中，并使用该工厂创建新对象。

```java
class ClassAsFactory<T> {
    T x;
    public ClassAsFactory(Class<T> kind) {
        try {
            x = kind.getDeclaredConstructor().newInstance();
        } catch(Exception e) {
            System.out.println(e);
        }
    }
}
class Employee {}


public class Test {
    public static void main(String[] args) {
        ClassAsFactory<Employee> fe = new ClassAsFactory(Employee.class);
    }
}
```

上面代码中使用了无参构造器，如果传入的类型没有无参构造器则无法工作。

```java
interface FactoryI<T> {
    T create();
}

class Foo<T> {
    private T x;
    public <F extends FactoryI<T>> Foo(F factory) {
    //public Foo(FactoryI factory) { // Error
        x = factory.create();
    }
}

class IntegerFactory implements FactoryI<Integer> {
    public Integer create() {
        return 0;
    }
}

class Widget {
    public static class Factory implements FactoryI<Widget> {
        public Widget create() {
            return new Widget();
        }
    }
}
public class Test {
    public static void main(String[] args) {
        new Foo<Integer>(new IntegerFactory());
        new Foo<Widget>(new Widget.Factory());
    }
}
```

使用了显式的工厂对象后，可以应用于任何一种类型，并且获得了编译期检查。

```java
abstract class GenericWithCreate<T> {
    final T element;
    GenericWithCreate() {
        element = create();
    }
    abstract T create();
}
class X {}

class Creator extends GenericWithCreate {
    X create() {
        return new X();
    }
}
public class Test {
    public static void main(String[] args) {
        Creator c = new Creator();
    }
}
```

上面代码使用了模板方法。

### 泛型数组

- 一般情况下使用`ArrayList`。如果确实需要数组，则只能使用强制类型转换`(T[])new Object[size]`。