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

### 使用wait()和notifyAll()进行线程协作

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

### 使用显式的Lock对象和Condition对象

- 可以使用显式的操作来进行同步与协作。使用互斥并允许任务挂起的基本类时`Condition`，可以通过在`Condition`上调用`await()`来挂起一个任务。当外部条件发生变化，意味着某个任务应该继续执行时，可以通过调用`signal()`来通知这个任务，从而唤醒一个任务，或者调用`signalAll()`来唤醒所有在这个`Condition`上被其自身挂起的任务。

```java
class Car {
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
    private boolean waxOn = false;

    public void waxed() {
        lock.lock();
        try {
            waxOn = true;
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void buffed() {
        lock.lock();
        try {
            waxOn = false;
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void waitForWaxing() throws InterruptedException {
        lock.lock();
        try {
            while (!waxOn)
                condition.await();
        } finally {
            lock.unlock();
        }
    }

    public void waitForBuffing() throws InterruptedException {
        lock.lock();
        try {
            while (waxOn)
                condition.await();
        } finally {
            lock.unlock();
        }
    }
}

class WaxOn implements Runnable {
    private Car car;

    public WaxOn(Car car) {
        this.car = car;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                System.out.println("Wax on");
                TimeUnit.MILLISECONDS.sleep(200);
                car.waxed();
                car.waitForBuffing();
            }
        } catch (InterruptedException e) {
            System.out.println("Exiting via interrupt");
        }
        System.out.println("Exiting Wax On task");
    }
}

class WaxOff implements Runnable {
    private Car car;

    public WaxOff(Car car) {
        this.car = car;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                car.waitForWaxing();
                System.out.println("Wax Off!");
                TimeUnit.MILLISECONDS.sleep(200);
                car.buffed();
            }
        } catch (InterruptedException e) {
            System.out.println("Exiting via interrupt");
        }
        System.out.println("Exiting Wax Off task");
    }
}

public class Main {

    public static void main(String[] args) throws InterruptedException {
        Car car = new Car();
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.execute(new WaxOff(car));
        exec.execute(new WaxOn(car));
        TimeUnit.SECONDS.sleep(5);
        exec.shutdownNow();
    }
}
```

- 单个`Lock`将产生一个`Condition`对象，这个对象用可以管理任务之间的通信。但是，`Condition`对象不含任何有关处理状态的信息，因此需要额外的变量来负责表示处理状态的信息，如上面代码中的`waxOn`。

### 生产者-消费者队列

- 同步队列时更高级别的抽象，同样可以解决任务协作问题，同步队列在任何时刻都只允许一个任务插入或移除元素。在`java.util.concurrent.BlockingQueue`接口中提供了这种队列。其实现`LinkedBlockingQueue`是一个无界队列，`ArrayBlockingQueue`则有固定尺寸。

```java
class Toast {
    public enum Status {DRY, BUTTERED, JAMMED}

    private Status status = Status.DRY;
    private final int id;

    public Toast(int idn) {
        id = idn;
    }

    public void butter() {
        status = Status.BUTTERED;
    }

    public void jam() {
        status = Status.JAMMED;
    }

    public Status getStatus() {
        return status;
    }

    public int getId() {
        return id;
    }

    public String toString() {
        return "Toast " + id + ": " + status;
    }
}

class ToastQueue extends LinkedBlockingQueue<Toast> {
}

class Toaster implements Runnable {
    private ToastQueue toasts;
    private int count = 0;
    private Random random = new Random(47);

    public Toaster(ToastQueue toasts) {
        this.toasts = toasts;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                TimeUnit.MILLISECONDS.sleep(100 + random.nextInt(500));
                Toast t = new Toast(count++);
                System.out.println(t);
                toasts.put(t);
            }
        } catch (InterruptedException e) {
            System.out.println("Toaster interrupted");
        }
        System.out.println("Toaster off ");
    }
}

class Butterer implements Runnable {
    private ToastQueue dryQueue, butteredQueue;

    public Butterer(ToastQueue dryQueue, ToastQueue butteredQueue) {
        this.dryQueue = dryQueue;
        this.butteredQueue = butteredQueue;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                Toast t = dryQueue.take();
                t.butter();
                System.out.println(t);
                butteredQueue.put(t);
            }
        } catch (InterruptedException e) {
            System.out.println("Butterer interrupted");
        }
        System.out.println("Butterer off");
    }
}

class Jammer implements Runnable {
    private ToastQueue butterQueue, finishedQueue;

    public Jammer(ToastQueue butterQueue, ToastQueue finishedQueue) {
        this.butterQueue = butterQueue;
        this.finishedQueue = finishedQueue;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                Toast t = butterQueue.take();
                t.jam();
                System.out.println(t);
                finishedQueue.put(t);
            }
        } catch (InterruptedException e) {
            System.out.println("Jammer interrupted");
        }
        System.out.println("Jammer off");
    }
}

class Eater implements Runnable {
    private ToastQueue finishedQueue;
    private int count = 0;

    public Eater(ToastQueue finishedQueue) {
        this.finishedQueue = finishedQueue;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                Toast t = finishedQueue.take();
                if (t.getId() != count++ || t.getStatus() != Toast.Status.JAMMED) {
                    System.out.println(">>>>>Error: " + t);
                    System.exit(1);
                } else System.out.println("Chomp! " + t);
            }
        } catch (InterruptedException e) {
            System.out.println("Eater interrupted");
        }
        System.out.println("Eater off");
    }
}

public class Main {

    public static void main(String[] args) throws InterruptedException {
        ToastQueue dryQueue = new ToastQueue(),
                butteredQueue = new ToastQueue(),
                finishedQueue = new ToastQueue();
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.execute(new Toaster(dryQueue));
        exec.execute(new Butterer(dryQueue, butteredQueue));
        exec.execute(new Jammer(butteredQueue, finishedQueue));
        exec.execute(new Eater(finishedQueue));
        TimeUnit.SECONDS.sleep(5);
        exec.shutdownNow();
    }
}
```

### 任务间使用管道进行输入输出

- 以管道的形式在线程间进行输入输出也很有用，在Java中对应为`PipedWriter`类（允许任务向管道写）和`PipedReader`类（允许不同任务从同一个管道中读）。

```java
class Sender implements Runnable {
    private Random random = new Random(47);
    private PipedWriter out = new PipedWriter();

    public PipedWriter getOut() {
        return out;
    }

    @Override
    public void run() {
        try {
            while (true) {
                for (char c = 'A'; c <= 'z'; c++) {
                    out.write(c);
                    TimeUnit.MILLISECONDS.sleep(500);
                }
            }
        } catch (IOException e) {
            System.out.println(e + " Sender write exception");
        } catch (InterruptedException e) {
            System.out.println(e + " Sender sleep interrupted");
        }
    }
}

class Receiver implements Runnable {
    private PipedReader in;

    public Receiver(Sender sender) throws IOException {
        in = new PipedReader(sender.getOut());
    }

    @Override
    public void run() {
        try {
            while (true) {
                System.out.println("Read: " + (char) in.read() + ", ");
            }
        } catch (IOException e) {
            System.out.println(e + " Receiver read exception");
        }
    }
}

public class Main {

    public static void main(String[] args) throws InterruptedException, IOException {
        Sender sender = new Sender();
        Receiver receiver = new Receiver(sender);
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.execute(sender);
        exec.execute(receiver);
        TimeUnit.SECONDS.sleep(4);
        exec.shutdownNow();
    }
}
```

