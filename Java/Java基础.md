---
title: Java基础
date: 2019-6-20 12:12:57
# img: /source/images/xxx.jpg 
top: false # 如果top值为true，则会是首页推荐文章
categories: 
- Java
tag:
- Java
- Java基础
---

# Java

## Java基础

### 1.数据类型

* 基本数据类型
  * byte/8
  * char/16
  * short/16
  * int/32
  * float/32
  * long/64
  * double/64
  * boolean/？
* 缓存池
  * new Integer(123)与 Integer.valueOf(123)的区别

### 2.String

* 不可变
  * 可以缓存hash值，不可变所以计算一次
* StringPool
  * 已建立的字符串将在StringPool引用
  * Java7以后，StringPool'被从永久代移动至堆，防止OOM

### 3.关键字

* final
  * 修饰类则表示类不可以被继承
  * 修饰方法表示方法不可被重写
  * 修饰基本类型表示不可变
  * 修饰对象表示引用不可变
* static
  * 修饰变量时静态变量通过类名访问，随类加载时加载
  * 修饰方法表示其随类加载而加载，不能是抽象方法
  * 修饰语句块表示随类加载运行一次
  * 静态内部类表示外部类被实例才可访问
  * 继承时的初始化顺序
    * 父类（静态变量、静态语句块）
    * 子类（静态变量、静态语句块）
    * 父类（实例变量、普通语句块）
    * 父类（构造函数）
    * 子类（实例变量、普通语句块）
    * 子类（构造函数）

### 4.Object方法

* hashCode()
* equals()
* toString()
* clone()
* wait()
* notify()
* notifyAll()
* finalize()
* getClass()

### 5.继承

* 抽象类不可被实例化，只能被继承
  * 使用抽象类的情况
    * 需要在几个相关的类中共享代码。
    * 需要能控制继承来的成员的访问权限，而不是都为 public。
    * 需要继承非静态和非常量字段
* Java8之前接口不可以有实现，默认字段都是static final
  * 使用接口的情况
    * 需要让不相关的类都实现一个方法，例如不相关的类都可以实现 Compareable 接口中的 compareTo() 方法；
    * 需要使用多重继承

* 重写与重载
  * 重写
    * 子类方法的访问权限必须大于等于父类方法；
    * 子类方法的返回类型必须是父类方法返回类型或为其子类型。
    * 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型。
  * 重载
    * 存在于同一个类中，指一个方法与已经存在的方法名称上相同，但是参数类型、个数、顺序至少有一个不同

### 6.反射、异常、泛型

* 反射
  * 通过获取字节码得到Class对象，可以控制类加载的时机
  * 可以通过Field，Method，Constructor分别获得成员变量，方法和创建新实例
* 异常
  * 继承Throwable的Error和Exception

### 7.新特性

* Stream

## Java容器

### Collection

* 各类集合容器通过模板编程实体类->抽象类->接口来设计，于此同时，为了实现基于反射的代理功能，实体类在继承抽象的同时也会实现接口，如果没有实现接口，在代理时就会类转换异常

#### Set

* TreeSet：基于红黑树实现底层使用TreeMap储存，不允许放入null值，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。
* HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。
  * 不能保证元素的排列顺序，顺序有可能发生变化
  * 不是同步的
  * 集合元素可以是null,但只能放入一个null
  * 要求放入的对象必须实现HashCode()方法，放入的对象，是以hashcode码作为标识的，而具有相同内容的 String对象，hashcode是一样，所以放入的内容不能重复。但是同一个类的对象可以放入不同的实例
* LinkedHashSet：具有 HashSet 的查找效率，并且内部使用双向链表维护元素的插入顺序。

#### List

* ArrayList：基于动态数组实现，支持随机访问。
  * 继承AbstractList，实现RandomAccess、Cloneable、Serializable等接口
  * 1.5倍扩容
  * transient关键字修饰elementData，保证未填满的数据不参与序列化，序列化通过readObject和writeObject实现
  * sort方法使用TimSort，是一个经过大量优化的归并+插入排序
* Vector：和 ArrayList 类似，但它是线程安全的。通过方法加sychronized
* LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。

#### Queue

* LinkedList：可以用它来实现双向队列。
* PriorityQueue：基于数组表示的小顶堆结构实现，可以用它来实现优先队列

#### Map

* TreeMap：基于红黑树实现。支持值null不支持键null

* HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程同时写入 HashTable 不会导致数据不一致。它是遗留类，不应该去使用它，而是使用 ConcurrentHashMap 来支持线程安全，ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁。

* LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。

* HashMap：基于哈希表实现。支持键值双null

  * 成员变量
    * 初始化桶大小，因为底层是数组，所以这是数组默认的大小。
    * 桶最大值。
    * 默认的负载因子（0.75）
    * TREEIFY_THRESHOLD 用于判断是否需要将链表转换为红黑树的阈值。
    * HashEntry 修改为 Node。
    * Map 存放数量的大小。
    * 桶大小，可在初始化时显式指定。
    * 负载因子，可在初始化时显式指定。
  * put方法
    1. 判断当前桶是否为空，空的就需要初始化（resize 中会判断是否进行初始化）。
    2. 根据当前 key 的 hashcode 定位到具体的桶中并判断是否为空，为空表明没有 Hash 冲突就直接在当前位置创建一个新桶即可。
    3. 如果当前桶有值（ Hash 冲突），那么就要比较当前桶中的 key、key 的 hashcode 与写入的 key 是否相等，相等就赋值给 e,在第 8 步的时候会统一进行赋值及返回。
    4. 如果当前桶为红黑树，那就要按照红黑树的方式写入数据。
    5. 如果是个链表，就需要将当前的 key、value 封装成一个新节点写入到当前桶的后面（形成链表）。
    6. 接着判断当前链表的大小是否大于预设的阈值，大于时就要转换为红黑树。
    7. 如果在遍历过程中找到 key 相同时直接退出遍历。
    8. 如果 e != null 就相当于存在相同的 key,那就需要将值覆盖。
    9. 最后判断是否需要进行扩容。
  * get方法
    1. 首先将 key hash 之后取得所定位的桶。
    2. 如果桶为空则直接返回 null 。
    3. 否则判断桶的第一个位置(有可能是链表、红黑树)的 key 是否为查询的 key，是就直接返回 value。
    4. 如果第一个不匹配，则判断它的下一个是红黑树还是链表。
    5. 红黑树就按照树的查找方式返回值。
    6. 不然就按照链表的方式遍历匹配返回值。
  * 2倍或是2的指数次幂扩容的原因
    * HashMap的容量为什么是2的n次幂，和这个(n - 1) & hash的计算方法有着千丝万缕的关系，符号&是按位与的计算，这是位运算，计算机能直接运算，特别高效，按位与&的计算方法是，只有当对应位置的数据都为1时，运算结果也为1，当HashMap的容量是2的n次幂时，(n-1)的2进制也就是1111111***111这样形式的，这样与添加元素的hash值进行位运算时，能够充分的散列，使得添加的元素均匀分布在HashMap的每个位置上，减少hash碰撞

* ConcurrentHashMap

  * 原理
    * 1.7ConcurrentHashMap 采用了分段锁技术，其中 Segment 继承于 ReentrantLock。不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment 数组数量)的线程并发。每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。
    * 1.8抛弃了原有的 Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性，HashEntry 改为 Node
  * 1.7put方法
    1. 将当前 Segment 中的 table 通过 key 的 hashcode 定位到 HashEntry。
    2. 遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value。
    3. 不为空则需要新建一个 HashEntry 并加入到 Segment 中，同时会先判断是否需要扩容。
    4. 最后会解除在 1 中所获取当前 Segment 的锁
  * 1.8put方法
    1. 根据 key 计算出 hashcode 。
    2. 判断是否需要进行初始化。
    3. f 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
    4. 如果当前位置的 hashcode == MOVED == -1,则需要进行扩容。
    5. 如果都不满足，则利用 synchronized 锁写入数据。
    6. 如果数量大于 TREEIFY_THRESHOLD 则要转换为红黑树
  * 1.7get方法
    1. 只需要将 Key 通过 Hash 之后定位到具体的 Segment ，再通过一次 Hash 定位到具体的元素上。
    2. 由于 HashEntry 中的 value 属性是用 volatile 关键词修饰的，保证了内存可见性，所以每次获取时都是最新值。
    3. ConcurrentHashMap 的 get 方法是非常高效的，因为整个过程都不需要加锁
  * 1.8get方法
    1. 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
    2. 如果是红黑树那就按照树的方式获取值。
    3. 就不满足那就按照链表的方式遍历获取值

* LinkedHashMap

  * 存储结构

    继承自 HashMap，因此具有和 HashMap 一样的快速查找特性。内部维护了一个双向链表，用来维护插入顺序或者 LRU 顺序。accessOrder 决定了顺序，默认为 false，此时维护的是插入顺序。LinkedHashMap 最重要的是以下用于维护顺序的函数，它们会在 put、get 等方法中调用* 

  * afterNodeAccess()

    当一个节点被访问时，如果 accessOrder 为 true，则会将该节点移到链表尾部。也就是说指定为 LRU 顺序之后，在每次访问一个节点时，会将这个节点移到链表尾部，保证链表尾部是最近访问的节点，那么链表首部就是最近最久未使用的节点。

  * afterNodeInsertion()

    在 put 等操作之后执行，当 removeEldestEntry() 方法返回 true 时会移除最晚的节点，也就是链表首部节点 first。evict 只有在构建 Map 的时候才为 false，在这里为 true。removeEldestEntry() 默认为 false，如果需要让它为 true，需要继承 LinkedHashMap 并且覆盖这个方法的实现，这在实现 LRU 的缓存中特别有用，通过移除最近最久未使用的节点，从而保证缓存空间足够，并且缓存的数据都是热点数据。

  * LRU 缓存
    * 设定最大缓存空间 MAX_ENTRIES  为 3；
    * 使用 LinkedHashMap 的构造函数将 accessOrder 设置为 true，开启 LRU 顺序；
    * 覆盖 removeEldestEntry() 方法实现，在节点多于 MAX_ENTRIES 就会将最近最久未使用的数据移除。

* WeakHashMap

  * 存储结构

    WeakHashMap 的 Entry 继承自 WeakReference，被 WeakReference 关联的对象在下一次垃圾回收时会被回收。

    WeakHashMap 主要用来实现缓存，通过使用 WeakHashMap 来引用缓存对象，由 JVM 对这部分缓存进行回收。

  * ConcurrentCache

    Tomcat 中的 ConcurrentCache 使用了 WeakHashMap 来实现缓存功能。ConcurrentCache 采取的是分代缓存：

    - 经常使用的对象放入 eden 中，eden 使用 ConcurrentHashMap 实现，不用担心会被回收（伊甸园）；
  - 不常用的对象放入 longterm，longterm 使用 WeakHashMap 实现，这些老对象会被垃圾收集器回收。
    - 当调用  get() 方法时，会先从 eden 区获取，如果没有找到的话再到 longterm 获取，当从 longterm 获取到就把对象放入 eden 中，从而保证经常被访问的节点不容易被回收。
    - 当调用 put() 方法时，如果 eden 的大小超过了 size，那么就将 eden 中的所有对象都放入 longterm 中，利用虚拟机回收掉一部分不经常使用的对象。

## JavaIO

### 分类概述

- 磁盘操作：File
- 字节操作：InputStream 和 OutputStream
- 字符操作：Reader 和 Writer
- 对象操作：Serializable
- 网络操作：Socket
- 新的输入/输出：NIO

### 磁盘操作

* 使用File类
  * File 类可以用于表示文件和目录的信息，但是它不表示文件的内容。

### 字节操作

* Java I/O 使用了装饰者模式来实现。以 InputStream 为例，
  * InputStream 是抽象组件；
  * FileInputStream 是 InputStream 的子类，属于具体组件，提供了字节流的输入操作；
  * FilterInputStream 属于抽象装饰者，装饰者用于装饰组件，为组件提供额外的功能。例如 BufferedInputStream 为 FileInputStream 提供缓存的功能。

### 字符操作

#### 编码与解码

> 编码就是把字符转换为字节，而解码是把字节重新组合成字符。如果编码和解码过程使用不同的编码方式那么就出现了乱码。

- GBK 编码中，中文字符占 2 个字节，英文字符占 1 个字节；
- UTF-8 编码中，中文字符占 3 个字节，英文字符占 1 个字节；
- UTF-16be 编码中，中文字符和英文字符都占 2 个字节。
  * UTF-16be 中的 be 指的是 Big Endian，也就是大端。相应地也有 UTF-16le，le 指的是 Little Endian，也就是小端。
  * Java 的内存编码使用双字节编码 UTF-16be，这不是指 Java 只支持这一种编码方式，而是说 char 这种类型使用 UTF-16be 进行编码。char 类型占 16 位，也就是两个字节，Java 使用这种双字节编码是为了让一个中文或者一个英文都能使用一个 char 来存储。

#### String 的编码方式

> String 可以看成一个字符序列，可以指定一个编码方式将它编码为字节序列，也可以指定一个编码方式将一个字节序列解码为 String。

```java
String str1 = "中文";
byte[] bytes = str1.getBytes("UTF-8");
String str2 = new String(bytes, "UTF-8");
System.out.println(str2);
```

> 在调用无参数 getBytes() 方法时，默认的编码方式不是 UTF-16be。双字节编码的好处是可以使用一个 char 存储中文和英文，而将 String 转为 bytes[] 字节数组就不再需要这个好处，因此也就不再需要双字节编码。getBytes() 的默认编码方式与平台有关，一般为 UTF-8。

```java
byte[] bytes = str1.getBytes();
```

#### Reader 与 Writer

> 不管是磁盘还是网络传输，最小的存储单元都是字节，而不是字符。但是在程序中操作的通常是字符形式的数据，因此需要提供对字符进行操作的方法。

- InputStreamReader 实现从字节流解码成字符流；
- OutputStreamWriter 实现字符流编码成为字节流。

### 对象操作

#### 序列化

* 序列化就是将一个对象转换成字节序列，方便存储和传输。
  * 序列化：ObjectOutputStream.writeObject()
  * 反序列化：ObjectInputStream.readObject()

* 不会对静态变量进行序列化，因为序列化只是保存对象的状态，静态变量属于类的状态。

#### Serializable

* 序列化的类需要实现 Serializable 接口，它只是一个标准，没有任何方法需要实现，但是如果不去实现它的话而进行序列化，会抛出异常。

### 网络操作

#### Java 中的网络支持

- InetAddress：用于表示网络上的硬件资源，即 IP 地址；
- URL：统一资源定位符；
- Sockets：使用 TCP 协议实现网络通信；
- Datagram：使用 UDP 协议实现网络通信。

#### InetAddress

没有公有的构造函数，只能通过静态方法来创建实例。

```java
InetAddress.getByName(String host);
InetAddress.getByAddress(byte[] address);
```

#### URL

可以直接从 URL 中读取字节流数据。

```java
public static void main(String[] args) throws IOException {
	URL url=new URL("http://www.baidu.com");
    BufferedReader reader=new BufferedReader(new InputStreamReader(url.openStream()));
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
    reader.close();
}
```

#### Sockets

- ServerSocket：服务器端类
- Socket：客户端类
- 服务器和客户端通过 InputStream 和 OutputStream 进行输入输出。

#### Datagram

- DatagramSocket：通信类
- DatagramPacket：数据包类

### NIO

> 新的输入/输出 (NIO) 库在 JDK 1.4 中引入，弥补了原来的 I/O 的不足，提供了高速的、面向块的 I/O

#### 流与块

* I/O 与 NIO 最重要的区别是数据打包和传输的方式，I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。
* 面向流的 I/O 一次处理一个字节数据：一个输入流产生一个字节数据，一个输出流消费一个字节数据。为流式数据创建过滤器非常容易，链接几个过滤器，以便每个过滤器只负责复杂处理机制的一部分。不利的一面是，面向流的 I/O 通常相当慢。
* 面向块的 I/O 一次处理一个数据块，按块处理数据比按流处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。
* I/O 包和 NIO 已经很好地集成了，java.io.\* 已经以 NIO 为基础重新实现了，所以现在它可以利用 NIO 的一些特性。例如，java.io.\* 包中的一些类包含以块的形式读写数据的方法，这使得即使在面向流的系统中，处理速度也会更快。

#### 通道

* 通道 Channel 是对原 I/O 包中的流的模拟，可以通过它读取和写入数据。
* 通道与流的不同之处在于，流只能在一个方向上移动(一个流必须是 InputStream 或者 OutputStream 的子类)，而通道是双向的，可以用于读、写或者同时用于读写。
* 通道包括以下类型：
  - FileChannel：从文件中读写数据；
  - DatagramChannel：通过 UDP 读写网络中数据；
  - SocketChannel：通过 TCP 读写网络中数据；
  - ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。

#### 缓冲区

* 发送给一个通道的所有数据都必须首先放到缓冲区中，同样地，从通道中读取的任何数据都要先读到缓冲区中。也就是说，不会直接对通道进行读写数据，而是要先经过缓冲区。
* 缓冲区实质上是一个数组，但它不仅仅是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。
* 缓冲区包括以下类型：
  - ByteBuffer
  - CharBuffer
  - ShortBuffer
  - IntBuffer
  - LongBuffer
  - FloatBuffer
  - DoubleBuffer
* 缓冲区状态变量
  * capacity：最大容量；
  * position：当前已经读写的字节数；
  * limit：还可以读写的字节数。

#### 选择器

* NIO 常常被叫做非阻塞 IO，主要是因为 NIO 在网络通信中的非阻塞特性被广泛使用。

* NIO 实现了 IO 多路复用中的 Reactor 模型，一个线程 Thread 使用一个选择器 Selector 通过轮询的方式去监听多个通道 Channel 上的事件，从而让一个线程就可以处理多个事件。

* 通过配置监听的通道 Channel 为非阻塞，那么当 Channel 上的 IO 事件还未到达时，就不会进入阻塞状态一直等待，而是继续轮询其它 Channel，找到 IO 事件已经到达的 Channel 执行。

* 因为创建和切换线程的开销很大，因此使用一个线程来处理多个事件而不是一个线程处理一个事件，对于 IO 密集型的应用性能更优。

* 应该注意的是，只有套接字 Channel 才能配置为非阻塞，而 FileChannel 不能，为 FileChannel 配置非阻塞也没有意义。

* 实例

  1. 创建选择器

     ```java
     Selector selector = Selector.open();
     ```

  2. 将通道注册到选择器上

     ```java
     ServerSocketChannel ssChannel = ServerSocketChannel.open();
     ssChannel.configureBlocking(false);
     ssChannel.register(selector, SelectionKey.OP_ACCEPT);
     ```
     
     通道必须配置为非阻塞模式，否则使用选择器就没有任何意义了，因为如果通道在某个事件上被阻塞，那么服务器就不能响应其它事件，必须等待这个事件处理完毕才能去处理其它事件，显然这和选择器的作用背道而驰。
     
     在将通道注册到选择器上时，还需要指定要注册的具体事件，主要有以下几类：
     
     - SelectionKey.OP_CONNECT
     - SelectionKey.OP_ACCEPT
     - SelectionKey.OP_READ
     - SelectionKey.OP_WRITE
     
     它们在 SelectionKey 的定义如下：
     
     ```java
     public static final int OP_READ = 1 << 0;
     public static final int OP_WRITE = 1 << 2;
     public static final int OP_CONNECT = 1 << 3;
     public static final int OP_ACCEPT = 1 << 4;
     ```
     
       可以看出每个事件可以被当成一个位域，从而组成事件集整数。例如：
     
     ```java
     int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
     ```
     
  3. 监听事件
  
     ```java
     int num = selector.select();
     ```
  
  4. 使用 select() 来监听到达的事件，它会一直阻塞直到有至少一个事件到达。
  
     获取到达的事件
  
     ```java
     Set<SelectionKey> keys = selector.selectedKeys();
     Iterator<SelectionKey> keyIterator = keys.iterator();
     while (keyIterator.hasNext()) {
         SelectionKey key = keyIterator.next();
         if (key.isAcceptable()) {
             // ...
         } else if (key.isReadable()) {
             // ...
         }
         keyIterator.remove();
     }
     ```
  
  5. 事件循环
  
     因为一次 select() 调用不能处理完所有的事件，并且服务器端有可能需要一直监听事件，因此服务器端处理事件的代码一般会放在一个死循环内。
  
     ```java
     while (true) {
         int num = selector.select();
         Set<SelectionKey> keys = selector.selectedKeys();
         Iterator<SelectionKey> keyIterator = keys.iterator();
         while (keyIterator.hasNext()) {
             SelectionKey key = keyIterator.next();
             if (key.isAcceptable()) {
                 // ...
             } else if (key.isReadable()) {
                 // ...
             }
             keyIterator.remove();
         }
     }
     ```

