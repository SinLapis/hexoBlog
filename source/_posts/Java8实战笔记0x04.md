---
title: Java8实战笔记0x04
date: 2019-08-12 21:05:45
categories: Java
tags:
  - Java
  - 笔记
---

# 重构、测试和调试

## 为改善可读性和灵活性重构代码

### 从匿名类到Lambda表达式的转换

- 将实现单一抽象方法的匿名类转换为Lambda表达式。

```java
Runnable r1 = new Runnable() {
    public void run() {
        System.out.println("Hello");
    }
};

Runnable r2 = () -> System.out.println("Hello");
```

转换时应当注意：
- 匿名类和Lambda表达式中的`this`和`super`的含义是不同的。在匿名类中，`this`代表的类是自身，但在Lambda中代表的是包含类（*外部类*）。
- 匿名类可以屏蔽包含类的变量，而Lambda表达式不能。

```java
int a = 10;
Runnable r1 = () -> {
    //! int a = 2;
    System.out.println(a);
};

Runnable r2 = new Runnable() {
    int a = 2; //正确
    System.out.println(a);
};
```

- 在涉及重载的上下文里，Lambda表达式将无法确定其类型，而匿名类则在初始化时确定了其类型。

```java
interface Task {
    public void execute();
}
public class Main {

    public static void test(Runnable runnable) {
        System.out.println("in runnable");
        runnable.run();
    }
    public static void test(Task task) {
        System.out.println("in task");
        task.execute();
    }
    public static void main(String[] args) {
        //! Main.test(() -> System.out.println("error"));
        // 无法通过编译
        // Error:(22, 13) java: 对test的引用不明确
        //     Main 中的方法 test(java.lang.Runnable) 和 Main 中的方法 test(Task) 都匹配
        Main.test((Task) () -> System.out.println("right"));
        // 使用强制转换，正确
    }
}
```

### 从Lambda表达式到方法引用的转换

- 为了改善代码的可读性，尽量使用方法引用，因为方法名往往能更直观地表达代码的意图。另外，还应该考虑使用静态辅助方法，比如`comparing`、`maxBy`。

### 从命令式的数据处理切换到Stream

- Stream API能更清晰地表达数据处理管道的意图。另外，通过短路和延迟载入以及利用多核，可以对Stream进行优化。但是这是一个困难的任务，因为要选择适当的流操作来还原控制流语句，例如`break`、`continue`以及`return`。

```java
List<String> dishName = new ArrayList<>();
for (Dish dish: menu) {
    if (dish.getCalories() > 300) {
        dishNames.add(dish.getName());
    }
}

menu.parallelStream()
    .filter(d -> d.getCalories() > 300)
    .map(Dish::getName)
    .collect(toList());
```

