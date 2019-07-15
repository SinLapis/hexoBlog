---
title: Java编程思想笔记0x10
date: 2019-07-15 15:36:47
categories: Java
tags:
  - Java
  - 笔记
---

# Java I/O 系统（六）

## 对象序列化

- Java的对象序列化将那些实现了`Serializable`接口的对象转换成一个字节序列，并能够在以后将这个字节序列完全恢复为原来的对象。这一过程甚至可通过网络进行；这意味着序列化机制能自动弥补不同操作系统之间的差异。
- 对象序列化可以实现轻量级持久性。持久性值为准一个对象的生存周期并不取决于程序是否在执行，而是可以生存与程序调用之间。之所以称其为轻量级，是因为不能用某种持久的关键字来简单地定义一个对象，并让系统自动维护其它细节问题，相反，对象必须在程序中显示地序列化和反序列化。
- 对象序列化可以用于Java的远程方法调用和Java Beans对象序列化。

> 实现RMI的主要步骤：
>
> 1. 定义一个远程接口，此接口需要继承Remote
> 2. 开发远程接口的实现类
> 3. 创建一个server并把远程对象注册到端口
> 4. 创建一个client查找远程对象，调用远程方法、

> Java Bean类是指Java中遵循关于命名、构造器、方法的特定规范的一种类型，以提供通用性。

- 只要对象实现了`Serializable`接口就可以进行对象序列化。`Serializable`是一个标记接口，其中没有任何方法。
- 序列化一个对象，首先要创建某些`OutputStream`对象，然后将其封装在一个`ObjectOutputStream`对象内，然后调用`writeObject()`即可将对象序列化，并将其发送给`OutputStream`；反序列化一个对象，需要将一个`InputStream`对象封装在`ObjectInputStream`中，然后调用`readObject()`，此时获得一个引用，指向一个向上转型的`Object`，需要进行合适的向下转型才能使用。
- 对象序列化不仅保存了对象本身的信息，而且能最终对象内所包含的所有引用，并保存那些对象；接着对这些对象继续追踪它们包含的引用并保存，以此类推。

### 寻找类

- 必须保证Java虚拟机能够找到序列化对象对应类的.class文件，否则会出现`ClassNotFoundException`。

```java
class Alien implements Serializable {}
public class Main {
    public static void main(String[] args) throws Exception {
        ObjectOutput out = new ObjectOutputStream(new FileOutputStream("x.file"));
        Alien a = new Alien();
        out.writeObject(a);
    }
}
```

此时再编写反序列化部分，和`Alien`不在同一目录下。

```java
public class Test {
    public static void main(String[] args) throws Exception {
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("x.file"));
        Object a = in.readObject();
        System.out.println(a.getClass());
    }
}
/* Output:
Exception in thread "main" java.lang.ClassNotFoundException: Alien
...
*/
```

*如果前后序列化的类名字相同，但是成员有变化会出现异常。*

```java
// 序列化时定义
class Alien implements Serializable {
    int i = 1;
}
```

```java
// 反序列化时定义
class Alien implements Serializable {
    String i = "1";
}
```

*此时进行反序列化会出现：*

```
Exception in thread "main" java.io.InvalidClassException: Alien; local class incompatible: stream classdesc serialVersionUID = 185057998004728250, local class serialVersionUID = 6323316111603380755
```

*但是如果成员名称和权限相同，成员变量仅赋值不同，成员方法的返回值和参数列表均相同但方法内部实现不同则不会出现异常，例如：*

```java
// 序列化时定义
class Alien implements Serializable {
    int k = 0;
    public int a() {
        int i = 0;
        i++;
        return i;
    }
}
```

```java
// 反序列化时定义
class Alien implements Serializable {
    int k = 566;
    public int a() {
        return 10;
    }
}
```

*此时进行反序列化不会出现异常。*

### 序列化控制

- 如果希望对序列化过程进行控制，可以通过实现`Externalizable`接口代替`Serializable`。

```java
public class Test {
    public static void main(String[] args) throws Exception {
        System.out.println("Constructing objects...");
        Blip1 b1 = new Blip1();
        Blip2 b2 = new Blip2();
        ObjectOutputStream o = new ObjectOutputStream(new FileOutputStream("blips.out"));
        System.out.println("Saving objects...");
        o.writeObject(b1);
        o.writeObject(b2);
        o.close();
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("blips.out"));
        System.out.println("Recovering b1...");
        b1 = (Blip1) in.readObject();
        System.out.println("Recovering b2...");
        b2 = (Blip2) in.readObject();
        in.close();;
    }
}
class Blip1 implements Externalizable {
    public Blip1() {
        System.out.println("Blip1 Constructor");
    }
    public void writeExternal(ObjectOutput out) throws IOException {
        System.out.println("Blip1#writeExternal");
    }
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        System.out.println("Blip1#reeadExternal");
    }
}
class Blip2 implements Externalizable {
    Blip2() {
        System.out.println("Blip2  Constructor");
    }
    public void writeExternal(ObjectOutput out) throws IOException {
        System.out.println("Blip2#writeExternal");
    }
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        System.out.println("Blip2#reeadExternal");
    }
}
/* Output:
Constructing objects...
Blip1 Constructor
Blip2  Constructor
Saving objects...
Blip1#writeExternal
Blip2#writeExternal
Recovering b1...
Blip1 Constructor
Blip1#reeadExternal
Recovering b2...
Exception in thread "main" java.io.InvalidClassException: Blip2; no valid constructor
*/
```

- `Externalizable`和`Serializable`不同，后者完全以对象存储的二进制位来构造，而不需要构造器，前者则会调用类所有默认的构造器（包括字段定义的初始化），然后调用`readExternal()`。

```java
public class Blip3 implements Externalizable {
    private int i;
    private String s;
    public Blip3() {
        System.out.println("Blip3 Constructor");
    }
    public Blip3(String x, int a) {
        s = x;
        i = a;
    }
    public String toString() {
        return s + i;
    }
    public void writeExternal(ObjectOutput out) throws IOException {
        System.out.println("Blip3#writeExternal");
        // 可以控制序列化成员
        out.writeObject(s);
        out.writeInt(i);
    }
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        // 按照writeExternal中写入顺序读出
        s = (String) in.readObject();
        i = in.readInt();
    }
    public static void main(String[] args) throws Exception {
        System.out.println("Constructing objects...");
        Blip3 b3 = new Blip3();
        System.out.println(b3);
        ObjectOutputStream o = new ObjectOutputStream(new FileOutputStream("b3.out"));
        System.out.println("Saving object...");
        o.writeObject(b3);
        o.close();
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("b3.out"));
        System.out.println("Recovering object...");
        b3 = (Blips3) in.readObject();
        System.out.println(b3);
    }
}
```

- 在上面代码中，如果在`writeExternal()`和`readExternal()`不保存和恢复成员变量，那么反序列化对象后，其成员变量值均为初始值（`i`为`0`，`s`为`null`）。
- 如果不希望对象中某个变量被序列化，除了实现`Externalizable`接口以外，可以使用`Serializable`配合关键字`transient`，该关键字修饰的成员变量在序列化时会被忽略。例如：

```java
private transient String password;
```

无论在使用中对`password`赋任何值，在序列化和反序列化后，其值一定为`null`。

- 尽管`Serializable`中没有定义方法，但是如果在实现该接口的类中添加`writeObject(ObjectOutputStream out)`和`readObject(ObjectInputStream in)`，那么对该类的对象进行序列化和反序列化时就会调用这两个方法，无论其权限如何（即使是`private`也会被调用）。此处使用的是反射搜索方法，而不是检查接口。

