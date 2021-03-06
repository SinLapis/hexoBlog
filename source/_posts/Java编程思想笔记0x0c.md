---
title: Java编程思想笔记0x0c
date: 2019-07-11 14:43:45
categories: Java
tags:
  - Java
  - 笔记
---

# Java I/O 系统（二）

## I/O流典型的使用方式

### 缓冲输入文件

```java
class BufferedInputFile {
    public static String read(String filename) throws IOException {
        BufferedReader in = new BufferedReader(new FileReader(filename));
        String s;
        StringBuilder sb = new StringBuilder();
        while((s = in.readLine()) != null) sb.append(s + "\n");
        in.close();
        return sb.toString();
    }
}
```

- 如果想要打开一个文件用于字符输入，可以使用以`String`或`File`对象作为文件名的`FileInputReader`。为了提高速度，需要对文件进行缓冲，可以将所产生的引用传给一个`BufferedReader`构造器。`BufferedReader`也提供`readLine()`方法，可以进行内容读取。当`readLine()`返回`null`时即达到文件末尾。最后调用`close()`关闭文件。

### 从内存输入

```java
public class Main {
    public static void main(String[] args) throws IOException {
        StringReader in = new StringReader("aaa bbb ccc ddd\n");
        int c;
        while((c = in.read()) != -1)
            System.out.print((char)c);
        in.close();
    }
}
```

- 使用`StringReader`读取内存中的`String`，`read()`每次会读取一个字节（是`int`形式，注意使用类型转换）。

### 格式化的内存输入

```java
public class Main {
    public static void main(String[] args) throws IOException {
        DataInputStream in = new DataInputStream(new BufferedInputStream(new FileInputStream("Main.java")));
        while (in.available() != 0) {
            System.out.println((char)in.readByte());
        }
        /* 一次读多个字节
        byte[] b = new byte[10];
        while (in.available() != 0) {
            in.read(b);
            System.out.println(Arrays.toString(b));
        }
        */
    }
}
```

- 读取格式化的数据，需要使用`DataInputStream`，这是一个面向字节的I/O类。因为任何字节的值都是合法结果，所以需要`available()`方法检查还有多少可供存取的字符。

### 基本的文件输出

```java
public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new FileReader("a.txt"));
        PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter("b.txt")));
        int lineCount = 1;
        String s;
        while((s = in.readLine()) != null)
            out.println(lineCount++ + ": " + s);
        in.close();
        out.close();
    }
}
```

- `FileWriter`对象可以向文件写入数据。首先，创建一个与指定文件连接的`FileWriter`。此外，通常会用`BufferedWriter`将其包装来缓冲输出，因为缓冲往往能够显著地提高I/O性能。为了提供格式化机制，使用了`PrintWriter`装饰。最后，应该显式调用`Writer#close()`，否则缓冲区内容不会刷新清空，内容也不会写入文件。
- 在Java SE5中，`PrintWriter`添加了一个辅助构造器，可以省略其它装饰器，直接填入文件名`String`即可。

### 存储和恢复数据

```java
public class Main {
    public static void main(String[] args) throws IOException {
        DataOutputStream out = new DataOutputStream(new BufferedOutputStream(new FileOutputStream("a.txt")));
        out.writeDouble(3.14159);
        out.writeUTF("That was pi");
        out.writeDouble(1.41413);
        out.writeUTF("Square root of 2");
        out.close();
        DataInputStream in = new DataInputStream(new BufferedInputStream(new FileInputStream("a.txt")));
        System.out.println(in.readDouble());
        System.out.println(in.readUTF());
        System.out.println(in.readDouble());
        System.out.println(in.readUTF());
    }
}
```

- 可以用`DataOutputStream`、`DataInputStream`以与机器无关方式从底层流中读写基本 Java 数据类型。
- 当使用`DataOutputStream`写字符串并让`DataInputStream`能够恢复它的唯一可靠做法就是使用UTF-8编码，在上面代码中就是使用`writeUTF()`和`readUTF()`。
- ASCII字符只占7位，为了避免空间浪费，UTF-8将ASCII字符编码成单一字节形式，而非ASCII字符则编码成两到三个字节的形式。另外，字符串的长度存储在UTF-8字符串的前两个字节。`writeUTF()`和`readUTF()`使用的是UTF-8变体。
- 为了保证所有的读方法能够正常工作，必须知道流中数据项所在的确切位置，要么为文件中的数据采用固定的割舍，标准要么将额外的信息保存在文件中以便能够对其解析来确定数据的存放位置。
