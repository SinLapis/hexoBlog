---
title: Java编程思想笔记0x16
date: 2019-07-30 20:24:33
categories: Java
tags:
  - Java
  - 笔记
---

# 并发（三）

## 线程之间的协作

### wait()与notifyAll()

- `wait()`会将任务挂起，并且只有`notify()`和`notifyAll()`发生时，即表示发生了某些感兴趣的食物，这个任务才会被唤醒并去检查所产生的变化。因此，`wait()`提供了一种在任务之间对活动同步对方式。
- 与`sleep()`的区别：
  1. 在`wait()`期间对象锁时释放的
  2. 可以通过`notify()`、`notifyAll()`，或者令时间到期，从`wait()`中恢复执行
- 调用`sleep()`、`yield()`时锁并不会释放。
- `wait()`、`notify()`、`notifyAll()`是基类`Object`的一部分，而不是`Thread`的一部分。
- `wait()`、`notify()`、`notifyAll()`可以并且只能同步控制方法或者同步控制块中调用，如果在非同步部分调用会得到`IllegalMonitorStateException`异常，即调用这些方法必须拥有（或者）对象的锁。

### 错失信号

```java
// T1
synchronized(sharedMonitor) {
    shareMonitor.notify();
}
// T2
while (trye) {
    synchronized(shareMonitor) {
        sharedMonitor.wait();
    }
}
```

- 在上面代码中，如果先执行了T1中的`notify()`后执行T2中的`wait()`，此时T2已经错失了T1发来的信号，从而产生了死锁。T2的改进方法见下面代码。

```java
synchronized (sharedMonitor) {
    while (someCondition) sharedMonitor.wait();
}
```

- 此时如果T1先执行，由于条件改变，T2就不会进入`wait()`。此外，这种方式还能防止被错误唤醒，如果被错误唤醒但还满足等待条件时会继续进入`wait()`。
- `notifyAll()`因某个特定锁而被调用时，只有等待这个锁的任务才会被唤醒。