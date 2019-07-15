---
title: Java编程思想笔记0x0e
date: 2019-07-14 10:40:21
categories: Java
tags:
  - Java
  - 笔记
---

# Java I/O 系统（四）

## 新I/O

- `java.nio.*`包中引入了新的Java I/O类库，其目的在于提高速度。而速度的提高来自于所使用的结构更接近于操作系统执行I/O的方式：通道和缓冲器。在交互过程中只需要和缓冲器交互，把缓冲器派送到通道。通道要么从缓冲器获得数据，要么向缓冲器发送数据。
- 唯一直接与通道交互的缓冲器是`ByteBuffer`，可以存储未加工字节的缓冲器。
- 引入NIO后，`FileInputStream`、`OutputStream`、`RandomAccessFile`被修改，可以产生`FileChannel`，用于操纵字节流。`java.nio.channels.Channels`类提供了可以在通道中产生`Reader`和`Writer`等字符模式类的方法。

```java
public class Test {
    public static void main(String[] args) throws IOException {
        // FileChannel fc = new FileOutputStream("test.out").getChannel();
        FileOutputStream fos = new FileOutputStream("test.out");
        FileChannel fc = fos.getChannel();
        fc.write(ByteBuffer.wrap("Some text".getBytes()));
        fc.close();
        fos.close();
        // fc = new RandomAccessFile("test.out", "rw").getChannel();
        RandomAccessFile raf = new RandomAccessFile("test.out", "rw");
        fc = raf.getChannel();
        fc.position(fc.size());
        fc.write(ByteBuffer.wrap("Some more".getBytes()));
        fc.close();
        raf.close();
        // fc = new FileInputStream("test.out").getChannel();
        FileInputStream fis = new FileInputStream("test.out");
        fc = fis.getChannel();
        ByteBuffer buff = ByteBuffer.allocate(1024);
        fc.read(buff);
        buff.flip();
        while(buff.hasRemaining())
            System.out.print((char)buff.get());
        fc.close();
        fis.close();
    }
}
```

- `getChannel()`会产生一个`FileChannel`，可以向它传送用于读写的`ByteBuffer`，并且可以锁定文件的某些区域用于独占式访问。
- 对于只读访问，必须显式地使用静态`allocate()`方法来分配`ByteBuffer`。`ByteBuffer`的大小关乎I/O的速度。使用`allocateDirect()`可以产生与操作系统有更高耦合性的直接缓冲器，以获得更高的速度。但是这种分配的开支会更大，并且具体实现与操作系统相关。

```java
public class ChannelCopy {
    private static final int BSIZE = 1024;
    public static void main(String[] args) throw Exception {
        if(args.length !=2) {
            System.out.println("arguments: sourcefile destfile");
            System.exit(1);
        }
        FileInputStream in = new FileInputStream(args[0]);
        FileOutputStream out = new FileOutputStream(args[1]);
        FileChannel fi = in.getChannel(), fo = out.getChannel();
        ByteBuffer buffer = ByteBuffer.allocate(BSIZE);
        while((in.readf(buffer)) != -1) {
            buffer.flip();
            fo.write(buffer);
            buffer.clear();
        }
    }
}
```



- 一旦调用`read()`来告知`FileChannel`向`ByteBuffer`存储字节，就必须调用缓冲器上的`flip()`，准备好让其它对象来读取其内容。如果打算继续使用缓冲器执行`read()`，则必须使用`clear()`。
- `FileChannel#read()`返回`-1`时即达到了输入的末尾。
- `transferTo()`和`transferFrom()`可以将一个通道和另一个通道相连。

### 转换数据

```java
public class Test {
    public static void main(String[] args) throws IOException {
        // 直接输入输出
        FileOutputStream fos = new FileOutputStream("data2.txt");
        FileChannel fc = fos.getChannel();
        fc.write(ByteBuffer.wrap("Some text".getBytes()));
        fc.close();
        fos.close();
        FileInputStream fis = new FileInputStream("data2.txt");
        fc = fis.getChannel();
        ByteBuffer bb = ByteBuffer.allocate(1024);
        fc.read(bb);
        bb.flip();
        System.out.println(bb.asCharBuffer());
        System.out.println("-----");
        bb.rewind();
        // 输出时解码
        String encoding = System.getProperty("file.encoding");
        System.out.println("encoding: " + encoding);
        System.out.println(Charset.forName(encoding).decode(bb));
        System.out.println("-----");
        fis.close();
        // 输入时编码
        fos = new FileOutputStream("data2.txt");
        fc = fos.getChannel();
        fc.write(ByteBuffer.wrap("Some text".getBytes("UTF-16BE")));
        fc.close();
        fos.close();
        fis = new FileInputStream("data2.txt");
        fc = fis.getChannel();
        bb.clear();
        fc.read(bb);
        bb.flip();
        System.out.println(bb.asCharBuffer());
        // 使用CharBuffer输出
        fos = new FileOutputStream("data2.txt");
        fc = fos.getChannel();
        bb = ByteBuffer.allocate(24);
        bb.asCharBuffer().put("Some text");
        fc.write(bb);
        fc.close();
        fis.close();
        // 使用CharBuffer输入
        fis = new FileInputStream("data2.txt");
        fc = fis.getChannel();
        bb.clear();
        fc.read(bb);
        bb.flip();
        System.out.println(bb.asCharBuffer());
        fc.close();
        fis.close();
        fos.close();
    }
}
/* Output:
卯浥⁴數
-----
encoding: UTF-8
Some text
-----
Some text
Some text
*/
```

- `java.nio.CharBuffer`有一个`toString()`方法，可以返回一个包含缓冲器所有字符的字符串。但是缓冲器中容纳的是普通的字节，要将其转为字符，要么在输入时对其进行编码，要么在将其从缓冲器输出时对其进行解码。

### 获取基本类型

- 向`ByteBuffer`插入基本类型数据，可以利用`asCharBuffer()`、`asShortBuffer()`等获得该缓冲器上的视图，然后使用视图的`put()`方法。注意使用`ShortBuffer`的`put()`方法时需要进行强制类型转换，而其它所有的数据缓冲器在使用`put()`方法时，不要进行转换。

### 视图缓冲器

- 视图缓冲器可以通过某个特定的基本数据类型的试穿查看其底层的`ByteBuffer`。此外还可以从`ByteBuffer`一次一个或者成批（数组）读取基本类型值。

```java
public class IntBufferDemo {
    public static void main(String[] args) {
        ByteBuffer bb = ByteBuffer.allocate(1024);
        IntBuffer ib = bb.asIntBuffer();
        ib.put(new int[]{11, 42, 47, 99, 143, 811, 1016});
        System.out.println(ib.get(3));
        System.out.println("-----");
        ib.put(3, 1811);
        // [11, 42, 47, 1811, 143, 811, 1016]
        ib.flip();
        while(ib.hasRemaining()) {
            int i = ib.get();
            System.out.println(i);
        }
    }
}
```

- 上面代码中，先用重载后的`put()`方法存储一个整数数组。`get()`和`put()`方法调用直接访问底层`ByteBuffer`中的某个整数位置。

> 小端：低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。
> 大端：高位字节排放在内存的低地址端，低位字节排放在内存的高地址端。

- `ByteBuffer`默认是大端形式存储数据，并且数据在网络中传送时也常常使用大端模式。可以使用带有参数`ByteOrder.BIG_ENDIAN`或`ByteOrder.LITTLE_ENDIAN`的`order()`方法改变`ByteBuffer`的字节排序方式。

### 缓冲器的细节

- `Buffer`由数据和四个索引组成。四个索引`mark`、`position`、`limit`、`capacity`可以高效地访问及操纵`Buffer`中的数据。

| 方法                | 功能                                                         |
| ------------------- | ------------------------------------------------------------ |
| `capacity()`        | 返回缓冲区的容量`capacity`                                   |
| `clear()`           | 清空缓冲区，将`position`设置为0，`limit`设置为容量`capacity`，可以用于覆写缓冲区 |
| `flip()`            | 将`limit`设为`position`，`position`设为0，用于准备从缓冲区读取已经写入的数据 |
| `limit()`           | 返回`limit`值                                                |
| `limit(int lim)`    | 设置`limit`值                                                |
| `mark()`            | 将`mark`设为`position`                                       |
| `position()`        | 返回`position`值                                             |
| `position(int pos)` | 设置`position`值                                             |
| `remaining()`       | 返回`limit - position`                                       |
| `hasRemaining()`    | 若有介于`position`和`limit`之间的元素，则返回`true`          |

### 内存映射文件

- 内存映射文件可以创建和修改因为过大而不能放入内存的文件。对应类为`MappedByteBuffer`。

```java
public class LarggeMappedFiles {
    static int length = 0x8FFFFFFF;
    public static void main(String[] args) throws Exception {
        RandomAccessFile raf = new RandomAccessFile("test.dat", "rw");
        MappedByteBuffer mbb = raf.getChannel().map(FileChannel.MapMode.READ_WRITE, 0, length);
        for (int i = 0; i < length; i++)
            mbb.put((byte)'x');
        System.out.println("Finished writing");
        for (int i = length / 2; i < length / 2 + 6; i++)
            System.out.println((char)mbb.get(i));
    }
}
```

> `MappedByteBuffer`主要是通过`FileChannel#map`方法，把文件映射到虚拟内存，并返回逻辑地址`address`，后续文件的读写操作都使用维护的虚拟内存地址和偏移进行操作。

### 文件加锁

- 文件加锁机制允许同步访问某个作为共享资源的文件，并且文件锁对其它的操作系统进程是可见的，因为Java的文件加锁直接映射到了本地操作系统的加锁工具。

```java
public class FileLocking {
    public static void main(String[] args) throws Exception {
        FileOutputStream fos = new FileOutputStream("file.txt");
        FileLock fl = fos.getChannel().tryLock();
        if (fl != null) {
            System.out.println("Locked File");
            TimeUnit.MILLISECONDS.sleep(100);
            fl.release();
            System.out.println("Released Lock");
        }
        fos.close();
    }
}
```

- 通过对`FileChannel`调用`tryLock()`或`lock`，就可以获得整个文件的`FileLock`。`tryLock()`是非阻塞式的，会尝试一次获得锁，如果失败则直接返回。`lock()`是阻塞式的，会阻塞进程直到获得锁，或调用`lock()`的线程中断，或调用`lock()`的通道关闭。Java虚拟机会自动释放锁或者关闭加锁的通道，也可以显式使用`FileLock#release()`释放锁。
- 可以对文件的一部分上锁，使用重载的`tryLock(long position, long size, boolean shared)`或`lock(long position, long size, boolean shared)`，其中加锁区域由`size - position`决定，第三个参数指明是否为共享锁。
- 对独占所或者共享锁的支持必须由底层的操作系统提供，如果操作系统不支持共享锁并为每一个请求都创建一个锁，那么就会使用独占锁。
- `SocketChannel`、`DatagramChannel`、`ServerSocketChannel`不需要加锁，因为这些通道是从单进程实体继承而来，通常不在两个进程之间共享网络socket。
- 对于映射文件，同样可以进行部分加锁，以便其它进程可以修改文件中未被加锁的部分，例如数据库。其方法和上述过程相同。