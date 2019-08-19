---
title: 剑指Offer面试题-实现单例模式
date: 2019-08-17 15:28:03
categories: 设计模式
tags:
  - Java
  - 设计模式
  - 单例模式
  - 剑指Offer
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

  其中3和4是可以重排序的。如果线程a执行顺序为1243，执行4后切换到线程b，此时单例变量不为空，线程b将获得一个没有初始化完成的对象。

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

- 该方式是线程安全的，因为JVM在执行类的初始化阶段，会获得一个可以同步多个线程对同一个类的初始化的锁。假设线程a先进行了初始化，那么线程b一直等待初始化锁。线程a执行类初始化，即使发生了重排序，也不会影响线程a的初始化。线程a初始化完后，释放锁。线程b获得初始化锁，发现`Singleton`对象已经初始化完毕，释放锁，不进行初始化，获得`Singleton`对象。
- *上面的静态内部类式单例是**不能**防止反射攻击的，网上有些博客说是可以，我认为是错误的。攻击代码也是一般思路，也可能有优化空间。详见下面代码。*

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

public class Main {
    public static void main(String[] args) throws IllegalAccessException, InvocationTargetException, InstantiationException, NoSuchFieldException {
        // 保存初始单例
        Singleton s = Singleton.getSingleton();
        // 获取Singleton构造器，并创建一个新的实例
        Constructor<?> c = Singleton.class.getDeclaredConstructors()[0];
        c.setAccessible(true);
        Singleton s2 = (Singleton)c.newInstance();
        // 获取内部类的构造器，并创建一个实例（可能不需要此步骤，没有测试）
        Constructor<?> in = Singleton.class.getDeclaredClasses()[0].getDeclaredConstructors()[0];
        in.setAccessible(true);
        Object ins = in.newInstance();
        // 获取单例成员变量
        Field f = ins.getClass().getDeclaredFields()[0];
        // 去除final修饰符（修改final成员通用方法）
        Field modifiersField = Field.class.getDeclaredField("modifiers");
        modifiersField.setAccessible(true);
        modifiersField.setInt(f, f.getModifiers() & ~Modifier.FINAL);
        // 修改单例成员的值为新实例
        f.setAccessible(true);
        f.set(ins, s2);
        // 比较
        System.out.println(s.equals(Singleton.getSingleton()));;
    }
}
/* Output:
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by Main (file:./testJava/out/production/testJava/) to field java.lang.reflect.Field.modifiers
WARNING: Please consider reporting this to the maintainers of Main
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
false
*/
```



### 线程不安全

```java
class Singleton {
    private static Singleton singleton = null;
    
    private Singleton() {
        
    }
    
    public static Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

- 仅适用于单线程。

### 对象锁

```java
class Singleton {
    private static Singleton singleton = null;
    
    private Singleton() {
        
    }
    
    public static synchronized Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

- 线程安全，但是每次调用`getInstance()`时都会锁住对象，引起线程阻塞，效率较低。

## 饿汉式单例

- 指在类初始化时就已经实例化。

### 使用类实现

```java
public class Singleton {
    private static Singleton singleton = new Singleton();
    
    private Singleton() {
        
    }
    
    public static Singleton getInstance() {
        return singleton;
    }
}
```

- 线程安全，JVM会保证类只加载一次。

### 使用枚举实现

```java
public enum Singleton {
    INSTANCE;
}
```

- 枚举实例只会初始化一次，因此可以保证是单例。
- 线程安全，同样JVM会保证枚举只加载一次。
- 枚举实现单例可以防止反射调用构造函数，枚举在反编译后是`abstact class`，无法实例化。并且在反射调用构造函数时会检查所调用的构造函数是否属于枚举类型的，若是则抛出异常。
- 枚举实现单例可以防止反序列化生成新的实例，因为枚举反序列化时则是通过`java.lang.Enum`的`valueOf`方法来根据名字查找枚举对象。

