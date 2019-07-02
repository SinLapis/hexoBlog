---
title: Java编程思想笔记0x05
date: 2019-07-01 20:19:43
categories: Java
tags:
  - Java
  - 笔记
---

# 类型信息

## Class对象

- Java使用`Class`对象来执行其RTTI。
- 类是程序的一部分，每个类都有一个`Class`对象
- 类加载过程（见[深入理解Java虚拟机笔记0x02](2019/06/23/深入理解Java虚拟机笔记0x02/)）
- `Class`对象仅在需要的时候才被加载
- `Class.forName(String className)`可以取得`Class`对象的引用，参数为一个包含目标类的文本名（区分大小写）。注意此方法会使类加载并初始化，作为对比，使用`ClassLoader`对象的`loadClass()`方法只会对类进行加载，而不会初始化。
- `Object`对象有方法`getClass()`可以获取相应`Class`对象引用。
- `Class#getSimpleName()`返回不含包名的类名，`Class#getCanonicalName()`返回全限定的类名。
- `Class#getSuperclass()`返回其基类的`Class`对象
- 关于`Class#newInstance()`：该方法已于Java 9声明废弃，使用`Class#getDeclaredConstructor().newInstance()`代替。

### 泛化的Class引用

- 可以使用泛型将`Class`引用所指向的`Class`对象的类型进行限定，使其变得更为具体 。
- `Class<?>`与`Class`：`Class`没有表现出是否要限制`Class`的意思，而`Class<?>`则表示此处`Class`的限制是无限制。
- 向`Class`引用添加泛型语法的原因仅仅是为了提供编译期类型检查。

### 转型

- `Class#cast()`可以将参数对象转换为`Class`引用的类型

## 类型检查

- 关键字`instanceof`：判断对象是否为某种类型，用法：`x instanceof ClassName`。只可将其与命名类型比较，不能与`Class`对象比较。
- `Class#isInstance()`判断参数引用是否为`Class`引用的实例。
- 使用`instanceof`或者`Class#isInstance()`进行的判断是对象是否为目标类或者目标类的子类，而使用`==`或者`equals()`则表示对象是否确切的是目标类，而不是目标类的子类或者其他类。

## 反射

- Java中使用`Class`类和`java.lang.reflect`共同支持反射。在类库中，`Constructor`用于创建新的对象，`get()`和`set()`方法用于修改与`Field`对象关联的字段，`invoke()`方法用于调用与`Method`对象关联的方法；此外，`getFields()`、`getMethods()`、`getConstructors()`方法可以分别获得表示字段、方法以及构造器对象的数组。
- RTTI与反射的区别：对于RTTI来说，编译器在编译时打开和检查.class文件，而对于反射来说，.class文件在编译时是不可获取的。