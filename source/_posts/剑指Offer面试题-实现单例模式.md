---
title: 剑指Offer面试题-实现单例模式
date: 2019-08-17 15:28:03
categories: 设计模式
tags:
  - Java
  - 设计模式
---

# 实现单例模式

## 懒汉式单例

- 懒汉式单例指第一次调用时才进行实例化。

### 双重检验锁

```java
class Singleton {
    private volatile static Singleton singleton = null;

    private Singleton() {

    }

    public static Singleton getSingleton() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

- 构造函数应当设为`private`，获取单例函数应设为`public static`。

- 单例变量应当声明为`volatile`，禁止指令重排序。由于初始化实例`singleton = new Singleton();`不是原子操作，而是分为4个步骤：

  1. 申请内存空间
  2. 初始化默认值
  3. 执行构造器方法
  4. 连接引用和实例

  其中3和4是可以重排序的。如果线程a执行顺序为1243，执行4后线程b

- `synchronized`上锁对象是`Singleton.class`。

- `synchronized`代码块中还应该进行一次对实例的判空，因为如果线程a通过了第一个判空后，切换到线程b一直执行，直到创建单例再切回线程a，此时线程a已经不需要再创建单例了。

- 可见性由`synchronized`保证。

### 静态内部类

```java
class Singleton {
    private static Singleton singleton = null;

    private Singleton() {

    }

    private static class StaticSingleton {
        private static final Singleton SINGLETON = new Singleton();
    }

    public static Singleton getSingleton() {
        return StaticSingleton.SINGLETON;
    }
}
```

- 

