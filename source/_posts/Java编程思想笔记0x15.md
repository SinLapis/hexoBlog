---
title: Java编程思想笔记0x15
date: 2019-07-28 15:48:16
categories: Java
tags:
  - Java
  - 笔记
---

# 并发（二）

## 共享受限资源

### 不正确地访问资源

```java
class Even {
    private int i = 0;

    public int next() {
        ++i; // [1]
        ++i;
        return i;
    }
}
```

- 上面代码中，如果正确执行，那么`next()`方法返回的一定是偶数。但是在并发的条件下，[1]处如果线程被挂起，被另外一个线程调用该对象的`next()`方法时则会返回奇数。

### 使用synchronize进行同步控制

```java
class Even {
    private int i = 0;

    public synchronized int next() {
        ++i; // [1]
        ++i;
        return i;
    }
}
```

- 使用`synchronized`关键字进行同步控制，应将涉及竞争的数据成员都声明为`private`，仅允许通过方法来访问这些成员。然后将相关的方法声明为`synchronized`。
- 所有对象都自动含有单一的锁（也成为监视器）。当在对象上调用其任意的`synchronized`方法的时候，此对象都被加锁，此时该对象上的其他`synchronized`方法只有等前一个方法调用完毕并释放了锁之后才能被调用。
- 一个任务可以多次获得对象的锁。JVM会跟踪加锁的次数，每当这个相同的任务在这个对象上获得锁时，相应的计数器会递增；每当任务离开一个`synchronized`方法，计数会递减；当计数为0时，锁被完全释放，此时其他任务可以使用此资源。
- 针对每个类，也有一个锁（作为类的`Class`对象的一部分），所以`synchronized static`方法可以在类的范围内防止对`static`数据的并发访问。

### 使用显式的Lock对象

- `Lock`对象必须被显式地创建、锁定和释放。相比起`synchronized`，代码缺乏优雅性，但是更加灵活；并且，如果使用`synchronized`时某些事物失败的将会抛出异常，没有机会进行清理工作，而使用`Lock`对象可以在`finally`子句中进行清理工作。

```java
class Even {
    private int i = 0;
    private Lock lock = new ReentrantLock();

    public int next() {
        lock.lock();
        try {
            ++i;
            ++i;
            return i;
        } finally {
            lock.unlock();
        }
    }
}
```

### 原子性和可视性

- 原子操作是不能被线程调度机制中断的操作，一旦操作开始，那么它一定在可能发生的上下文切换之前执行完毕。

> 当变量声明为`volatile`时，变量将具备以下两个特性：
>
> 1. 保证此变量对所有线程的可见性，即一个线程修改了volatile变量后，其余线程可以立即获得修改后的值。
>
> 2. 禁止指令重排序优化，即设置内存屏障，保证volatile变量修改更新到所有CPU上。

- 在非`volatile`上的原子操作不必刷新到主存中去，因此其他读取该域的任务也不必看到这个新值。
- 如果多个任务在同时访问某个域，那么这个域就应该是`volatile`的，否则，这个域就应该只能经由同步来访问。同步也会导致向主存中刷新，因此如果一个域完全由`synchronized`方法或语句块来防护，那就不必将其设置为是`volatile`的。
- 当一个域的值依赖于它之前的值，或者受到其他域的值的限制时，`volatile`将无法工作。