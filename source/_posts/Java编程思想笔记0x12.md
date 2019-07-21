---
title: Java编程思想笔记0x12
date: 2019-07-21 09:22:48
categories: Java
tags:
  - Java
  - 笔记
---

# 枚举类型（二）

## 使用EnumSet替代标志

- 是一种面向`enum`的标志，可以用来表示某种“开/关”信息。其内部实现是将一个`long`值作为位向量，因此`EnumSet`非常高效。

```java
enum AlarmPoints {
    STAIR1, STAIR2, LOBBY, OFFICE1, OFFICE2, OFFICE3,
    OFFICE4, BATHROOM, UTILITY, KITCHEN
}

public class Test {
    public static void main(String[] args) throws Exception {
        EnumSet<AlarmPoints> points = EnumSet.noneOf(AlarmPoints.class);
        points.add(AlarmPoints.BATHROOM);
        System.out.println(points);
        points.addAll(EnumSet.of(AlarmPoints.STAIR1, AlarmPoints.STAIR2, AlarmPoints.KITCHEN));
        System.out.println(points);
        points = EnumSet.allOf(AlarmPoints.class);
        points.removeAll(EnumSet.of(AlarmPoints.STAIR1, AlarmPoints.STAIR2, AlarmPoints.KITCHEN));
        System.out.println(points);
        points.removeAll(EnumSet.range(AlarmPoints.OFFICE1, AlarmPoints.OFFICE4));
        System.out.println(points);
        points = EnumSet.complementOf(points);
        System.out.println(points);
    }
}
```

- <del>`EnumSet.of()`方法被重载了很多次，不仅为可变数量参数进行类重载，还为接收1到5个显式的参数的情况进行类重载。即当使用2到5个参数时，会调用对应的重载方法，而使用1个或者多于5个参数时，会调用可变参数的`of()`方法。注意，1个参数不会导致编译器构造可变参数数组。</del>
- *在JDK 11中，`EnumSet.of()`不同的重载方法有：1至5个显式声明参数的方法，和一个参数列表为`(E first, E... rest)`的方法。*

- *在创建`EnumSet`对象时，如果枚举实例小于等于64个，则使用一个`long`值，否则使用一个`long`数组。*

## 使用EnumMap

- `EnumMap`要求其中的键必须来自一个`enum`。由于枚举实例总数量是确定的，`EnumMap`内部使用了数组实现。调用`put()`方法时键只能是枚举实例，其它操作和普通的`Map`基本一致。

```java
enum AlarmPoints {
    STAIR1, STAIR2, LOBBY, OFFICE1, OFFICE2, OFFICE3,
    OFFICE4, BATHROOM, UTILITY, KITCHEN
}
interface Command {
    void action();
}

public class Test {
    public static void main(String[] args) throws Exception {
        EnumMap<AlarmPoints, Command> em = new EnumMap<>(AlarmPoints.class);
        em.put(AlarmPoints.KITCHEN, new Command() {
            public void action() {
                System.out.println("Kichen fire!");
            }
        });
        em.put(AlarmPoints.BATHROOM, new Command() {
            public void action() {
                System.out.println("Bathroom alart!");
            }
        });
        for(Map.Entry<AlarmPoints, Command> e: em.entrySet()) {
            System.out.print(e.getKey() + ": ");
            e.getValue().action();
        }
        try {
            em.get(AlarmPoints.UTILITY).action();
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
/* Output:
BATHROOM: Bathroom alart!
KITCHEN: Kichen fire!
java.lang.NullPointerException
*/
```

- 上面代码是一种命令模式的用法，即首先需要一个只有单一方法的接口，然后从该接口实现具有各自不同的行为的多个子类。之后可以根据需要构造命令对象并使用。
- 如果`EnumMap`没有为某个键调用`put()`方法来存入相应的值的话，那么其对应的值就为`null`。
- *注意`foreach`中元素的泛型声明不可省略*

## 常量相关方法

- 可以为`enum`实例编写方法，从而为每个枚举实例赋予各自不同的行为。首先需要为`enum`定义一个或多个方法（或者抽象方法），然后为选择枚举实例进行覆盖方法（如果是抽象方法则每个枚举实例都要实现该方法）。

```java
public enum Test {
    DATE_TIME {
        String getInfo() {
            return DateFormat.getDateInstance().format(new Date());
        }
    }, 
    CLASSPATH {
        String getInfo() {
            return System.getenv("CLASSPATH");
        }
    }, 
    VERSION {
        String getInfo() {
            return System.getProperty("java.version");
        }
    },
    TEST {
        String getInfo() {
            return "test info";
        }
        int getNumber() {
            return 222222;
        }
    };
    abstract String getInfo();
    int getNumber() {
        return 111111;
    }
    public static void main(String[] args) throws Exception {
        for (Test t: values()) {
            System.out.println(t.getInfo() + " " + t.getNumber());
        }
    }
}
/* Output:
2019年7月21日 111111
.;C:\Program Files\Java\jdk-11.0.3\lib\dt.jar;C:\Program Files\Java\jdk-11.0.3\lib\tools.jar; 111111
11.0.3 111111
test info 222222
*/
```

- 通过相应的枚举实例，可以调用其上的方法，这通常也称为表驱动的代码（tabledriven code）。在上面代码中，枚举实例似乎被当做其“超类”来使用，在调用覆盖（实现）的方法（抽象方法）时体现多态的行为。

### 使用enum职责链

- 在职责链设计模式中，可以用多种不同的方法来解决一个问题，然后将它们链接在一起。当一个请求到来时，遍历这个脸，直到链中的某个解决方案能够处理该请求。
- 通过常量相关的方法，可以很容易地实现一个简单的职责链。其中每一次尝试可以看做一个策略（设计模式），完整的处理方式列表就是一个职责链。

```java
enum Handle {
    PLUS {
        boolean modify(int src, int target) {
            switch(target - src) {
                case 1: 
                    System.out.println("plus");
                    return true;
                default: return false;
            }
        }
    },
    MINUS {
        boolean modify(int src, int target) {
            switch(target - src) {
                case -1: 
                    System.out.println("minus");
                    return true;
                default: return false;
            }
        }
    },
    ABS {
        boolean modify(int src, int target) {
            switch(target + src) {
                case 0: 
                    System.out.println("abs");
                    return true;
                default: return false;
            }
        }
    };
    abstract boolean modify(int src, int target);
} 
public class Test {
    static void modify(int src, int target) {
        System.out.print(src + " ");
        for(Handle h: Handle.values()) {
            if(h.modify(src, target)) return;
        }
        System.out.println("can't modify");
    }
    public static void main(String[] args) throws Exception {
        Random r = new Random(47);
        int target = r.nextInt(10) - 5;
        System.out.println(target);
        for (int i = 0; i < 10; i++) {
            int src = r.nextInt(10) - 5;
            modify(src, target);
        }
    }
}
/* Output:
3
0 can't modify
-2 can't modify
-4 can't modify
-4 can't modify
4 minus
3 can't modify
-5 can't modify
-3 abs
2 plus
3 can't modify
*/
```

### 使用enum的状态机

- 枚举类型非常适合用来创建状态机。一个状态机有有限的特定状态，它通常根据输入，从一个状态转移到下一个状态。每个状态都具有某些可接受的输入，不同的输入回事状态机从当前状态转移到不同的新状态。由于`enum`对其实例有严格的限制，非常适合用来表现不同的状态和输入。

```java
enum Check {
    EVEN {
        void next(char n) {
            switch(n) {
                case '0':
                    c = ODD;
                    return;
                case '1':
                    return;
                default:
                    c = ERROR;
            }
        }
    },
    ODD {
        void next(char n) {
            switch(n) {
                case '0':
                    c = EVEN;
                    return;
                case '1':
                    return;
                default:
                    c = ERROR;
            }
        }
    },
    ERROR;
    private static Check c;
    void next(char n) { throw new RuntimeException("no such method."); };
    public static void check(String num) {
        c = EVEN;
        char[] ns = num.toCharArray();
        for (char n: ns) {
            c.next(n);
            if(c == ERROR) break;
        }
        switch(c) {
            case ODD:
                System.out.println("odd 0s");
                return;
            case EVEN:
                System.out.println("even 0s");
                return;
            case ERROR:
                System.out.println("incorrect input");
        }
    }
}
public class Test {
    public static void main(String[] args) throws Exception {
        Check.check("01001100001");
        Check.check("000011110000");
        Check.check("00110020");
    }
}
/* Output:
odd 0s
even 0s
incorrect input
*/
```

- *上面代码中实现了判断一串字符串的二进制数中有奇数还是偶数个0。有三种状态，`EVEN`、`ODD`、`ERROR`，前两种状态根据输入的不同决定将要转移的下一个状态，而`ERROR`则表示输入有误，仅作为最终状态。*

## 多路分发

- *如果存在一种多元运算，其变量类型范围确定但是不能知道某个变量的具体类型，并且该运算结果与变量类型相关，即该运算的实现需要判断各个参数的类型，那么此时可以使用多路分发。*

```java
enum Outcome { WIN, DRAW, LOSE}
interface Item {
    Outcome compete(Item it);
    Outcome eval(Paper p);
    Outcome eval(Scissors s);
    Outcome eval(Rock r);
}
class Paper implements Item {
    public Outcome compete(Item it) { return it.eval(this); }
    public Outcome eval(Paper p) { return Outcome.DRAW; }
    public Outcome eval(Scissors s) { return Outcome.WIN; }
    public Outcome eval(Rock r) { return Outcome.LOSE; }
    public String toString() { return "Paper"; }
}
class Scissors implements Item {
    public Outcome compete(Item it) { return it.eval(this); }
    public Outcome eval(Paper p) { return Outcome.LOSE; }
    public Outcome eval(Scissors s) { return Outcome.DRAW; }
    public Outcome eval(Rock r) { return Outcome.WIN; }
    public String toString() { return "Scissors"; }
}
class Rock implements Item {
    public Outcome compete(Item it) { return it.eval(this); }
    public Outcome eval(Paper p) { return Outcome.WIN; }
    public Outcome eval(Scissors s) { return Outcome.LOSE; }
    public Outcome eval(Rock r) { return Outcome.DRAW; }
    public String toString() { return "Rock"; }
}
public class Test {
    static final int SIZE = 10;
    private static Random r = new Random(47);
    public static Item newItem() {
        switch(r.nextInt(3)) {
            default:
            case 0: return new Scissors();
            case 1: return new Rock();
            case 2: return new Paper();
        }
    }
    public static void match(Item a, Item b) {
        System.out.println(a + " vs. " + b + ": " + a.compete(b));
    }
    public static void main(String[] args) throws Exception {
        for (int i = 0; i < SIZE; i++) {
            match(newItem(), newItem());
        }
    }
}
```

- 上面代码实现了“石头、剪刀、布”游戏，采用了接口的方式进行多路分发（两路分发），避免了需要判断类型的代码。

### 使用enum分发

```java
enum Outcome { WIN, DRAW, LOSE}
enum RoShamBo2 {
    PAPER(Outcome.DRAW, Outcome.LOSE, Outcome.WIN),
    SCISSORS(Outcome.WIN, Outcome.DRAW, Outcome.LOSE),
    ROCK(Outcome.LOSE, Outcome.WIN, Outcome.DRAW);
    private Outcome vPaper, vScissors, vRock;
    RoShamBo2(Outcome paper, Outcome scissors, Outcome rock) {
        this.vPaper = paper;
        this.vScissorc = scissors;
        this.vRock = rock;
    }
    public Outcome compete(RoShamBo2 it) {
        switch(it) {
            default:
            case Outcome.PAPER: return vPaper;
            case Outcome.SCISSORS: return vScissors;
            case Outcome.ROCK: return vRock;
        }
    }
}
// 调用略
```

- 上面代码使用构造器来初始化每个枚举实例，并以一组结果作为参数，形成类类似查询表的结构。

### 使用常量相关的方法

- 为每个枚举实例提供方法的不同实现同样可以完成多路分发，但是在需要添加类型时代码的改动量要多于使用`enum`分发。

```java
enum Outcome { WIN, DRAW, LOSE}
enum RoShamBo3 {
    PAPER {
        public Outcome compete(RoShamBo3 it) {
            switch(it) {
                default: 
                case PAPER: return DRAW;
                case SCISSORS: return LOSE;
                case ROCK: return WIN;
            }
        }
    };
    // SCISSORS, ROCK略 
}
```

