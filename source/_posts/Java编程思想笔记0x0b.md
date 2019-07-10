---
title: Java编程思想笔记0x0b
date: 2019-07-09 20:14:59
categories: Java
tags:
  - Java
  - 笔记
---

# Java I/O 系统

## File

- `File`类既代表一个特定文件的名称，又能代表一个目录下的一组文件的名称。

  ```java
  public class Main {
      public static void main(String[] args) {
          File path = new File(".");
          String[] list;
          if(args.length == 0)
              list = path.list();
          else 
              list = path.list(new DirFilter(args[0]));
          Arrays.sort(list, String.CASE_INSENSITIVE_ORDER);
          for(String dirItem: list)
              System.out.println(dirItem);
      }
  }
  
  class DirFilter implements FilenameFilter {
      private Pattern pattern;
      public DirFilter(String regex) {
          pattern = Pattern.compile(regex);
      }
      @Override
      public boolean accept(File dir, String name) {
          return pattern.matcher(name).matches();
      }
  }
  ```

  - 上面代码中，`File#list()`可以接受一个`FilenameFilter`对象，用于过滤文件。此处使用了回调，重载了`accept()`方法并让`list()`调用。

  ## 输入和输出

  ### InputStream类型

  - `InputStream`的作用是用来表示那些从不同数据源产生的输入的类，包括：字节数组、`String`对象、文件、管道、由其他种类的流组成的序列等，每一种数据源都有相应的`InputStream`子类。

| 类                        | 功能                                                         | 构造器参数                                                   | 如何使用                                                     |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `ByteArrayInputStream`    | 允许将内存的缓冲区当做`InputStream`使用                      | 缓冲区，字节将从中取出                                       | 作为一种数据源：将其与`FilterInputStream`对象相连以提供有用接口 |
| `StringBufferInputStream` | 将`String`转换成`InputStream`使用                            | 字符串，底层是实现使用`StringBuffer`                         | 作为一种数据源：将其与`FilterInputStream`对象相连以提供有用接口 |
| `FileInputStream`         | 用于从文件中读取信息                                         | 字符串，表示文件名、文件或`FileDescriptor`对象               | 作为一种数据源：将其与`FilterInputStream`对象相连以提供有用接口 |
| `PipedInputStream`        | 产生用于写入相关`PipedOutputStream`的数据，实现管道化的概念  | `PipedOutputStream`                                          | 作为一种数据源：将其与`FilterInputStream`对象相连以提供有用接口 |
| `SequenceInputStream`     | 将两个或多个`InputStream`对象转换成单一`InputStream`         | 两个`InputSteam`对象或者一个容纳`InputStream`对象的容器`Enumeration` | 作为多线程中数据源：将其与`FilterInputStream`对象相连以提供有用接口 |
| `FilterInputStream`       | 抽象类，作为装饰器的接口。其中，装饰器为其它的`InputStream`类提供有用的功能 | -                                                            | -                                                            |

### OutputStream类型

- `OutputStream`类的类别决定了输出所要去往的目标：字节数组、文件或管道。

| 类                      | 功能                                                         | 构造器参数                                     | 如何使用                                                     |
| ----------------------- | ------------------------------------------------------------ | ---------------------------------------------- | ------------------------------------------------------------ |
| `ByteArrayOutputStream` | 在内存中创建缓冲区，所有送往流的数据都要放置在此缓冲区。     | 缓冲区初始化尺寸（可选）                       | 用于指定数据的目的地：将其与`FilterOutputStream`对象相连以提供有用接口 |
| `FileOutputStream`      | 用于将信息写至文件                                           | 字符串，表示文件名、文件或`FileDescriptor`对象 | 用于指定数据的目的地：将其与`FilterOutputStream`对象相连以提供有用接口 |
| `PipedOutputStream`     | 任何写入其中的信息都会自动作为相关`PipedInputStream`的输出。实现管道化的概念 | `PipedInputStream`                             | 用于指定多线程的数据的目的地：将其与`FilterOutputStream`对象相连以提供有用接口 |
| `FilterOutputStream`    | 抽象类，作为装饰器的皆苦。其中，装饰器为其它`OutputStream`提供有用功能 | -                                              | -                                                            |

## 添加属性和有用的接口

### FilterInputStream

| 类                      | 功能                                                         | 构造器参数                        | 如何使用                                             |
| ----------------------- | ------------------------------------------------------------ | --------------------------------- | ---------------------------------------------------- |
| `DataInputStream`       | 与`DataOutputStream`搭配使用，可以按照可移植方式从流读取基本数据类型（`int`、`char`、`long`等） | `InputStream`                     | 包含用于读取基本类型数据的全部接口                   |
| `BufferedInputStream`   | 可以防止每次读取时都得进行实际写操作，代表使用缓冲区。       | `InputStream`，可选指定缓冲区大小 | 本质上不提供接口，只不过是向进程中添加缓冲区所必须的 |
| `LineNumberInputStream` | 跟踪输入流中的行号，可调用`getLineNumber()`和`setLineNumber(int)` | `InputStream`                     | 仅增加了行号，因此可能要与接口对象搭配使用           |
| `PushbackInputStream`   | 能弹出一个字节的缓冲区，因此可以将读到的最后一个字符回退。   | `InputStream`                     | 通常作为编译器的扫描器                               |

### FilterOutputStream

| 类                     | 功能                                                         | 构造器参数                                   | 如何使用                                             |
| ---------------------- | ------------------------------------------------------------ | -------------------------------------------- | ---------------------------------------------------- |
| `DataOutputStream`     | 与`DataInputStream`搭配使用，可以按照可移植方式从流读取基本数据类型（`int`、`char`、`long`等） | `OutputStream`                               | 包含用于写入基本类型数据的全部接口                   |
| `PrintStream`          | 用于产生格式化输出。其中`DataOutputStream`处理数据的存储，`PrintStream`处理显示 | `OutputStream`，可选是否每次换行时清空缓冲区 | -                                                    |
| `BufferedOutputStream` | 可以防止每次读取时都得进行实际写操作，代表使用缓冲区。       | `OutputStream`，可选指定缓冲区大小           | 本质上不提供接口，只不过是向进程中添加缓冲区所必须的 |

## Reader和Writer

- `InputStream`和`OutputStream`是面向字节形式的I/O，而`Reader`和`Writer`则提供兼容Unicode与面向字符的I/O功能。

### 数据的来源与去处

| `InputStream`和`OutputStream` | `Reader`和`Writer`                    |
| ----------------------------- | ------------------------------------- |
| `InputStream`                 | `Reader`，适配器`InputStreamReader`   |
| `OutputStream`                | `Writer`， 适配器`OutputStreamWriter` |
| `FileInputStream`             | `FileReader`                          |
| `FileOutputStream`            | `FileWriter`                          |
| -                             | `StringReader`                        |
| -                             | `StringWriter`                        |
| `ByteArrayInputStream`        | `CharArrayReader`                     |
| `ByteArrayOutputStream`       | `CharArrayWriter`                     |
| `PipedInputStream`            | `PipedReader`                         |
| `PipedOutputStream`           | `PipedWriter`                         |

### 更改流的行为

| `FilterInputStream`和`FilterOutputStream` | `FilterReader`和`FilterWriter` |
| ----------------------------------------- | ------------------------------ |
| `FilterInputStream`                       | `FilterReader`                 |
| `FilterOutputStream`                      | `FilterWriter`                 |
| `BufferedInputStream`                     | `BufferedReader`               |
| `BufferedOutputStream`                    | `BufferedWriter`               |
| `PrintStream`                             | `PrintWriter`                  |

- 如果需要使用`readLine()`方法，就不应该使用`DataInputStream`，而用`BufferedReader`代替。

> `DataInputStream#readLine()`方法已被废弃，因为它无法正确地将字节转换为字符。

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

