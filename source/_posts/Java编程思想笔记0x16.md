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

### 生产者和消费者

```java
class Meal {
    private final int orderNum;

    public Meal(int orderNum) {
        this.orderNum = orderNum;
    }

    @Override
    public String toString() {
        return "Meal " + orderNum;
    }
}

class WaitPerson implements Runnable {
    private Restaurant restaurant;

    public WaitPerson(Restaurant restaurant) {
        this.restaurant = restaurant;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                synchronized (this) {
                    while (restaurant.meal == null)
                        wait(); 
                }
                System.out.println("Waitperson got " + restaurant.meal);
                synchronized (restaurant.chef) {
                    restaurant.meal = null;
                    restaurant.chef.notifyAll();
                }
            }
        } catch (InterruptedException e) {
            System.out.println("WaitPerson interrupted");
        }
    }
}

class Chef implements Runnable {
    private Restaurant restaurant;
    private int count = 0;

    public Chef(Restaurant restaurant) {
        this.restaurant = restaurant;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                synchronized (this) {
                    while (restaurant.meal != null)
                        wait();
                }
                if (++count == 10) {
                    System.out.println("Out of food, closing");
                    restaurant.exec.shutdownNow();
                }
                System.out.println("Order up!");
                synchronized (restaurant.waitPerson) {
                    restaurant.meal = new Meal(count);
                    restaurant.waitPerson.notifyAll();
                }
                TimeUnit.MILLISECONDS.sleep(100);
            }
        } catch (InterruptedException e) {
            System.out.println("Chef interrupted");
        }
    }
}

class Restaurant {
    public Meal meal;
    ExecutorService exec = Executors.newCachedThreadPool();
    WaitPerson waitPerson = new WaitPerson(this);
    Chef chef = new Chef(this);
    public Restaurant() {
        exec.execute(chef);
        try {
            TimeUnit.SECONDS.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        exec.execute(waitPerson);
    }
}

public class Main {

    public static void main(String[] args) throws InterruptedException {
        new Restaurant();
    }
}
```

- *上面代码使用`wait()`和`notifyAll()`的主要原因是减少资源竞争，从而降低CPU资源使用。实际上单生产者单消费者在不考虑资源使用的情况下，是没有必要加锁的。*
- 上面代码中只有一个任务，理论上可以调用`notify()`而不是`notifyAll()`。但是，在更复杂的情况下，可能会有多个任务在某个特定的对象锁上等待，因此无法知道哪个任务应该被唤醒。因此，调用`notifyAll()`要更安全一些，这样可以唤醒等待这个锁的所有任务，而每个任务都必须决定这个通知是否与自己相关。（*因为获取对象锁之后，线程的行为并不确定，因此应当使用`notifyAll()`来唤醒所有线程，每个线程来检查自己是否应当被唤醒。*）