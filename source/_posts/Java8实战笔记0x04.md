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

## 使用Lambda重构面向对象的设计模式

### 策略模式

- 策略模式代表了解决一类算法的通用解决方案，可以在运行时选择使用哪种方案。策略模式包含三部分内容：
  - 一个代表某个算法的接口。
  - 一个或多个该接口的具体实现，它们代表了算法的多种实现。
  - 一个或多个使用策略对象的客户。

```java
interface ValidationStrategy {
    boolean execute(String s);
}

class IsAllLowerCase implements ValidationStrategy {
    @Override
    public boolean execute(String s) {
        return s.matches("[a-z]+");
    }
}

class IsNumberic implements ValidationStrategy {
    @Override
    public boolean execute(String s) {
        return s.matches("\\d+");
    }
}

class Validator {
    private final ValidationStrategy strategy;
    
    public Validator(ValidationStrategy strategy) {
        this.strategy = strategy;
    }
    
    public boolean validate(String s) {
        return strategy.execute(s);
    }
}

public class Main {
    public static void main(String[] args) {
        Validator nv1 = new Validator(new IsNumberic());
        boolean b1 = nv1.validate("aaa");
        Validator lv1 = new Validator(new IsAllLowerCase());
        boolean b2 = lv1.validate("bbbb");
        
        Validator nv2 = new Validator((String s) -> s.matches("[a-z]+"));
        b1 = nv2.validate("aaa");
        Validator lv2 = new Validator((String s) -> s.matches("\\d+"));
        b2 = lv2.validate("bbb");
    }
}
```

### 模版方法

- 如果需要采用某个算法的框架，同时又希望有一定的灵活度，能对它的某些部分进行改进，那么采用模版方法设计模式是比较通用的方案。

```java
abstract class OnlineBanking {
    public void processCustomer(int id) {
        Custormer c = Database.getCustormerWithId(id);
        makeCustomerHappy(c);
    }
    abstract void makeCustomerHappy(Customer c);
}
```

- 上面代码搭建的在线银行算法框架，不同的支行可以通过继承`OnlineBanking`来提供不同的服务。

```java
class OnlineBanking {
    public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
        makeCustomerHappy.accept(c);
    }
}
//...
new OnlineBanking().processCustomer(1337, (Customer c) -> System.out.println("Hello!"));
```

### 观察者模式

- 观察者模式是，某些事件发生时（比如状态转变），如果一个对象（主题）需要自动地通知其他多个对象（观察者）。

```java
// 观察者接口
interface Observer {
    void notify(String tweet);
}
// 不同的观察者
class NYTimes implements Observer {
    public void notify(String tweet) {
        if (tweet != null && tweet.contains("money")) {
            System.out.println("Breaking news in NY! " + tweet);
        }
    }
}

class Guardian implements Observer {
    public void notify(String tweet) {
        if (tweet != null && tweet.contains("queen")) {
            System.out.println("Yet another news in London... " + tweet);
        }
    }
}

class LeMonde implements Observer {
    public void notify(String tweet) {
        if (tweet != null && tweet.contains("wine")) {
            System.out.println("Today cheese, wine and news! " + tweet);
        }
    }
}
// 主题接口
interface Subject {
    void registerObserver(Observer o);
    void notifyObservers(String tweet);
}
class Feed implements Subject {
    private final List<Observer> observers = new ArrayList<>();
    
    public void registerObserver(Observer o) {
        this.observers.add(o);
    }
    
   public void notifyObservers(String tweet) {
       observers.forEach(o -> o.notify(tweet));
   }
}
//...
// 使用
Feed f = new Feed();
f.registerObserver(new NYTimes());
f.registerObserver(new Guardian());
f.registerObserver(new LeMonde());
f.notifyObservers("The queen said her favourite book is Java 8 in Action!");

// 优化观察者声明和注册
f.registerObserver((String tweet) -> {
    if (tweet != null && tweet.contains("money")) {
            System.out.println("Breaking news in NY! " + tweet);
        }
});
//...
```

- 上面示例仅限简单的观察者模式。如果观察者的逻辑十分复杂，或者持有状态、定义了多个方法等等，此时应该继续使用类的方式。

### 责任链模式

- 责任链模式是一种创建处理对象序列的通用方案。一个处理对象可能需要在完成一些工作之后，将结果传递给另一个对象，这个对象接着做一些工作，再转交给下一个处理对象，以此类推。

```java
// 处理抽象类
abstract class ProcessingObject<T> {
    protected ProcessingObject<T> successor;
    public void setSuccessor(ProcessingObject<T> successor) {
        this.successor = successor;
    }
    public T handle(T input) {
        T r = handleWork(input);
        if (successor != null) {
            return successor.handle(r);
        } 
        return r;
    }
    abstract protected T handleWork(T input);
}
// 处理类
class HeaderTextProcessing extends ProcessingObject<String> {
    public String handleWord(String text) {
        return "From Raoul, Mario and Alan: " + text;
    }
}

class SpellCheckerProcessing extends ProcessingObject<String> {
    public String handleWord(String text) {
        return text.replaceAll("labda", "lambda");
    }
}
// 使用
ProcessingObject<String> p1 = new HeaderTextProcessing();
ProcessingObjct<String> p2 = new SpellCheckerProcessing();
p1.setSuccessor(p2);
String result = p1.handle("Aren't labdas really sexy?!!");
System.out.println(result);

// 使用Lambda优化处理对象实现和连接
UnaryOperator<String> headerProcessing = (String text) -> "From Raoul, Mario and Alan: " + text;
UnaryOperator<String> spellCheckerProcessing = (String text) -> text.replaceAll("labda", "lambda");
Function<String, String> pipeline = headerProcessing.andThen(spellCheckerProcessing);
String result = pipeline.apply("Aren't labdas really sexy?!!");
```

