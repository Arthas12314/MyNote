# JUC

笔记源课程：java并发编程与高并发解决方案

文档主要转载摘录自blog：https://blog.51cto.com/zero01/

## 概述

- 并发：多个线程操作相同的资源，保证线程安全，合理使用资源
- 高并发：服务能同时处理很多请求，提高程序性能
- CPU多级缓存：缓解CPU和内存之间速度的不匹配问题
	* 缓存一致性MESI 保证多个CPU cache之间缓存共享数据的一致
		+ M: 被修改（Modified) 
		该缓存行只被缓存在该CPU的缓存中，并且是被修改过的（dirty),即与主存中的数据不一致，该缓存行中的内存需要在未来的某个时间点（允许其它CPU读取请主存中相应内存之前）写回（write back）主存。
		+ E: 独享的（Exclusive)
		该缓存行只被缓存在该CPU的缓存中，它是未被修改过的（clean)，与主存中数据一致。该状态可以在任何时刻当有其它CPU读取该内存时变成共享状态（shared)。同样地，当CPU修改该缓存行中内容时，该状态可以变成Modified状态，当被写回主存之后，该缓存行的状态会变成独享（exclusive)状态
		+ S: 共享的（Shared)
		该状态意味着该缓存行可能被多个CPU缓存，并且各个缓存中的数据与主存数据一致（clean)，当有一个CPU修改该缓存行中，其它CPU中该缓存行可以被作废（变成无效状态（Invalid））
		+ I: 无效的（Invalid）
		该缓存是无效的（可能有其它CPU修改了该缓存行）
	* 乱序执行优化：处理器为提高运行速度做出违背代码原有顺序的优化

![MESI缓存一致性](C:\Users\hawk4\Desktop\临时\笔记\笔记图片\MESI缓存一致性.png)

## JMM
- JMM规范：它规定了一个线程如何和何时能够看到由其他线程修改过后的共享变量值以及在必须时如何同步地访问共享变量

![java内存模型](C:\Users\hawk4\Desktop\临时\笔记\笔记图片\java内存模型.jpg)

- 同步八种操作
	* lock 作用于主内存的变量，把一个变量标识为一条线程独占状态
	* unlock 作用于主内存的变量，把一个处于锁定状态的变量释放出来，释放后的变量才可已被其他线程锁定
	* read 作用于主内存的变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
	* load 作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中
	* use 作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎
	* assign 作用于工作内存的变量，他把一个从执行引擎收到的值赋值给工作内存的变量
	* store 作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write操作
	* write 作用于工作内存的变量，他把store操作从工作内存中一个变量的值传送到主内存的变量中
- 同步规则
	* 不允许read和load、store和write单独出现。即不允许一个变量从主内存读取了但工作内存不接受，或者从工作内从发起会写了但主内存不接受的情况出现。
	* 不允许一个线程丢弃它的最近的assign操作，即变量在工作内存中改变了之后必须把该变化同步到主内存里。
	* 不允许一个线程无原因地把数据从线程工作内存同步回主内存中，即没有发生过任何的assign操作就同步到主内存中。
	* 一个新的变量只能在主内存中诞生，不允许在工作内存中直接使用一个没有被初始化（load或assign）的变量，换句话说，就是对一个变量实施use、store操作之前，必须先执行assign和load操作。
	* 一个变量在同一时刻，只允许一个线程对其进行loack操作，但lock操作可以被同一个线程重复执行多次，多次执行lock后，只有执行相同次数的unloack操作，变量才能被解锁。
	* 如果对一个变量执行了lock操作，那将会清空工作内存中此变量的值，在执行引擎使用这个变量前，需要重新执行load或assign操作初始化变量的值。
	* 如果一个变量事先没有被lock操作锁定，那就不允许对他执行unlock操作，也不允许去unlock一个被其他线程锁定住的变量
	* 对一个变量执行unlock操作之前，必须先把此变量同步回主内存中（执行store、write操作）

![同步操作于规则](C:\Users\hawk4\Desktop\临时\笔记\笔记图片\同步操作于规则.jpg)

## 并发编程与线程安全
### 线程安全性
- 概念：
  当多个线程访问某个类时，不管运行时环境采用何种调度方式，并且在主调代码中不需要额外的同步，这个类都能表现出正确的行为，那么这个类就是线程安全的

- 原子性
提供了互斥访问，同一时刻只能有一个线程来对它进行操作
	* AtomicXXX：CAS U.weakCompareAndSetInt() 实现方式是通过循环判断底层值是否被其他线程进行了更改，若未更改则继续 由于更改失败情况下将循环判断，会浪费资源 适用于低并发下
	```java
	int v;
        do {
            v = getIntVolatile(o, offset);
        } while (!weakCompareAndSetInt(o, offset, v, v + delta));
        return v;
	```
	* AtomicLong: LongAdder() Hash算法分散至各节点提高并行，可能损失精度，高并发下效率高
	* AtomicReference：CAS
	```
	AtomicReference<Integer> count = new AtomicReference<>(0);
	count.compareAndSet(0, 2); // 2
    count.compareAndSet(0, 1); // no
    count.compareAndSet(1, 3); // no
    count.compareAndSet(2, 4); // 4	
	```
	* AtomicIntegerFieldUpdater

	```
	public class AtomicExample5 {
    private static AtomicIntegerFieldUpdater<AtomicExample5> updater =
            AtomicIntegerFieldUpdater.newUpdater(AtomicExample5.class, "count");
    
    public volatile int count = 100;

    public static void main(String[] args) {

        AtomicExample5 example5 = new AtomicExample5();

        if (updater.compareAndSet(example5, 100, 120)) {
            log.info("update success 1, {}", example5.getCount());
        }

        if (updater.compareAndSet(example5, 100, 120)) {
            log.info("update success 2, {}", example5.getCount());
        } else {
            log.info("update failed, {}", example5.getCount());
        }
    }
	}
	```
	
	* AtomicStampRedference CAS的ABA问题，解决思路是通过将版本号+1来计数判断，多了一个Stamp变量
	
	* AtomicBoolean
	
	  ```java
	  public class AtomicExample6 {
	  
	      private static AtomicBoolean isHappened = new AtomicBoolean(false);
	  
	      // 请求总数
	      public static int clientTotal = 5000;
	  
	      // 同时并发执行的线程数
	      public static int threadTotal = 200;
	  
	      public static void main(String[] args) throws Exception {
	          ExecutorService executorService = Executors.newCachedThreadPool();
	          final Semaphore semaphore = new Semaphore(threadTotal);
	          final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
	          for (int i = 0; i < clientTotal ; i++) {
	              executorService.execute(() -> {
	                  try {
	                      semaphore.acquire();
	                      test();
	                      semaphore.release();
	                  } catch (Exception e) {
	                      log.error("exception", e);
	                  }
	                  countDownLatch.countDown();
	              });
	          }
	          countDownLatch.await();
	          executorService.shutdown();
	          log.info("isHappened:{}", isHappened.get());
	      }
	  
	      private static void test() {
	          if (isHappened.compareAndSet(false, true)) {
	              log.info("execute");
	          }
	      }
	  }
	  ```
	
	* Synchronize ：不可中断锁，适合竞争不激烈的情况，可读性好
	  + 修饰代码块作用于调用的对象
	  + 修饰方法作用于调用的对象
	  + 修饰静态方法作用于所有对象
	  + 修饰类作用于所有对象
	
	* Lock：可中断锁，多样化同步，竞争激烈时能维持常态
	* Atomic：竞争激烈时能维持常态，比Lock性能好，只能同步一个值
	
- 可见性
  一个线程对主内存的修改可以及时的被其他线程观察到

  * 导致共享变量在线程间不可见的原因
    + 线程交叉执行
    + 重排序结合线程交叉执行
    + 共享变量更新后的值没有在工作内存与主存间及时更新

  * JMM关于synchronized规定
    * 线程解锁前，必须把共享变量的最新值刷新到主内存
    * 线程加锁时，将清空工作内存中共享变量的值，从而使用共享变量时需要从主内存中重新读取最新的值（枷锁和解所时同一把锁

  * volatile 

    一般用于变量不依赖当前值，且变量没有包含在具有其他变量不必要的式子中，适合用于状态标记量

    + 对volatile变量写操作时，会在写操作后加入一条store屏障指令，将本地内存中的共享变量值刷新到主内存
    + 对volatile变量读操作时，会在读操作前加入一条load屏障指令，从主内存中读取共享变量

- 有序性
  一个线程观察其他线程中的指令执行顺序，由于指令重排的存在，该观察结果一般杂乱无序

  在Java内存模型中，允许编译器和处理器对指令进行重排序，但是重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性
  
  * happens-before原则（先行发生原则）
  
    如果两个操作的次序不能从这八种规则中推导出来，则不能保证有序性
  
    + 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生书写在后面的操作
    + 锁定规则：一个unLock操作先行发生于后面对同一个锁的lock操作
    + volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作
    + 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C
    + 线程启动规则：Thread对象的start() 方法先行发生于此线程的每一个动作
    + 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
    + 线程终结规则：线程中所有操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束，可以通过Thread.isAlive()的返回值检测到线程已经终止执行
    + 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始

## 发布对象

- 发布对象：使一个对象能够被当前范围之外的代码所使用，例如通过方法返回对象的引用，或者通过公有的静态变量发布对象

- 对象逸出：一种错误的发布，当一个对象还没有构造完成时，就使它被其他线程所见

- 不正确的发布可变对象导致的两种错误：

  * 发布线程意外的所有线程都可以看到被发布对象的过期的值

  * 线程看到的被发布对象的引用是最新的，然而被发布对象的状态却是过期的

### 安全发布对象

#### 四个线程安全策略

- 线程限制：一个被线程限制的对象，由线程独占，并且只能被占有它的线程修改
- 共享只读：一个共享只读的对象，在没有额外同步的情况下，可以被多个线程并发访问，但是任何线程都不能修改它
- 线程安全对象：一个线程安全的对象或者容器，在内部通过同步机制来保证线程安全，所以其他线程无需额外的同步就可以通过公共接口随意访问它
- 被守护对象：被守护对象只能通过获取特定的锁来访问

#### 安全发布方法

- 在静态初始化函数中初始化一个对象的引用
- 将对象的引用保存到volatile类型域或者AtomicReference对象中
- 将对象的引用保存到某个正确构造对象的final类型域中
- 将对象的引用保存到一个由锁保护的域中

#### 不可变对象

- final关键字
  * 修饰类：不能被继承（final类中的所有方法都会被隐式的声明为final方法）
  * 修饰方法：
    - 锁定方法不被继承类修改；
    - 可提升效率（private方法被隐式修饰为final方法）
  * 修饰变量：基本数据类型变量（初始化之后不能修改）、引用类型变量（初始化之后不能再修改其引用）
  * 修饰方法参数：同修饰变量
- 通常我们会使用一些工具类来完成不可变对象的创建：
  - Collections.unmodifiableXXX：Collection、List、Set、Map...
  - Guava：ImmutableXXX：Collection、List、Set、Map...

### 线程封闭

线程封闭最常见的应用就是用在数据库连接对象上，数据库连接对象本身并不是线程安全的，但由于线程封闭的作用，一个线程只会持有一个连接对象，并且持有的连接对象不会被其他线程所获取，这样就不会有线程安全的问题了

- 概念：把对象封装到一个线程里，只有这个线程能看到这个对象。那么即便这个对象本身不是线程安全的，但由于线程封闭的关系让其只能在一个线程里访问，所以也就不会出现线程安全的问题了
- 实现方式：
  * Ad-hoc 线程封闭：完全由程序控制实现，最糟糕的方式，忽略
  * 堆栈封闭：局部变量，当多个线程访问同一个方法的时候，方法内的局部变量都会被拷贝一份副本到线程的栈中，所以局部变量是不会被多个线程所共享的，因此无并发问题。所以我们在开发时应尽量使用局部变量而不是全局变量
  * ThreadLocal 线程封闭：每个Thread线程内部都有个map，这个map是以线程本地对象作为key，以线程的变量副本作为value。而这个map是由ThreadLocal来维护的，由ThreadLocal负责向map里设置线程的变量值，以及获取值。所以对于不同的线程，每次获取副本值的时候，其他线程都不能获取当前线程的副本值，于是就形成了副本的隔离，多个线程互不干扰。所以这是特别好的实现线程封闭的方式

#### 常见的线程不安全的类与写法

所谓线程不安全的类，是指该类的实例对象可以同时被多个线程共享访问，如果不做同步或线程安全的处理，就会表现出线程不安全的行为

- 字符串拼接，在Java里提供了两个类可完成字符串拼接，就是StringBuilder和StringBuffer，其中StringBuilder是线程不安全的，而StringBuffer是线程安全的
  * StringBuffer之所以是线程安全的原因是几乎所有的方法都加了synchronized关键字，所以是线程安全的。但是由于StringBuffer 是以加 synchronized 这种暴力的方式保证的线程安全，所以性能会相对较差，在堆栈封闭等线程安全的环境下应该首先选用StringBuilder
- SimpleDateFormat
  * SimpleDateFormat 的实例对象在多线程共享使用的时候会抛出转换异常，正确的使用方法应该是采用堆栈封闭，将其作为方法内的局部变量而不是全局变量，在每次调用方法的时候才去创建一个SimpleDateFormat实例对象，这样利于堆栈封闭就不会出现并发问题。另一种方式是使用第三方库joda-time的DateTimeFormatter类(推荐使用)
- ArrayList, HashMap, HashSet 等 Collections 都是线程不安全的
- **有一种写法需要注意，即便是线程安全的对象，在这种写法下也可能会出现线程不安全的行为，这种写法就是先检查后执行**

```Java
if(condition(a)){
    handle(a);
}
```

### 同步容器简介

- 在Java中同步容器主要分为两类，一类是集合接口下的同步容器实现类

  * List -> Vector、Stack

  * Map -> HashTable（key、value不能为null）

  * vector的所有方法都是有synchronized关键字保护的，stack继承了vector，并且提供了栈操作（先进后出），而hashtable也是由synchronized关键字保护

    但是需要注意的是同步容器也并不一定是绝对线程安全的，例如有两个线程，线程A根据size的值循环执行remove操作，而线程B根据size的值循环执行执行get操作。它们都需要调用size获取容器大小，当循环到最后一个元素时，若线程A先remove了线程B需要get的元素，那么就会报越界错误

- 第二类是Collections.synchronizedXXX (list,set,map)方法所创建的同步容器

### 并发容器

- ArrayList对应的CopyOnWriteArrayList

  * CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。而在CopyOnWriteArrayList写的过程是会加锁的，即调用add的时候，否则多线程写的时候会Copy出N个副本出来

    读的时候不需要加锁，但是如果读的时候有多个线程正在向CopyOnWriteArrayList添加数据，读还是会读到旧的数据，因为写的时候不会锁住旧的CopyOnWriteArrayList

  * CopyOnWriteArrayList容器有很多优点，但是同时也存在两个问题，即内存占用问题和数据一致性问题

    + 内存占用问题

      因为CopyOnWriteArrayList的写操作时的复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象（注意:在复制的时候只是复制容器里的引用，只是在写的时候会创建新对象添加到新容器里，而旧容器的对象还在使用，所以有两份对象内存）。如果这些对象占用的内存比较大，比如说200M左右，那么再写入100M数据进去，内存就会占用300M，那么这个时候很有可能造成频繁的Yong GC和Full GC。之前我们系统中使用了一个服务由于每晚使用CopyOnWrite机制更新大对象，造成了每晚15秒的Full GC，应用响应时间也随之变长。

      针对内存占用问题，可以通过压缩容器中的元素的方法来减少大对象的内存消耗，比如，如果元素全是10进制的数字，可以考虑把它压缩成36进制或64进制。或者不使用CopyOnWrite容器，而使用其他的并发容器，如ConcurrentHashMap

    + 数据一致性问题

      CopyOnWriteArrayList容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，即实时读取场景，那么请不要使用CopyOnWriteArrayList容器

  * CopyOnWrite的应用场景

    CopyOnWriteArrayList并发容器用于读多写少的并发场景。在数据较多的情况下，每次add/set都要重新复制数组，这个代价过于高昂。在高性能的互联网应用中，这种操作极易引起故障

- HashSet对应的CopyOnWriteArraySet

  CopyOnWriteArraySet是线程安全的，它底层的实现使用了CopyOnWriteArrayList，因此和CopyOnWriteArrayList概念是类似的。使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不可变的数组快照，所以迭代器不支持可变的 remove 操作

  * CopyOnWriteArraySet适合于具有以下特征的场景

    set 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突

  * CopyOnWriteArraySet缺点

    因为通常需要复制整个基础数组，所以可变操作（add、set 和 remove 等等）的开销很大

- TreeSet对应的ConcurrentSkipListSet

  * ConcurrentSkipListSet是jdk6新增的类，它和TreeSet一样是支持自然排序的，并且可以在构造的时候定义`Comparator<E>` 的比较器，该类的方法基本和TreeSet中方法一样（方法签名一样）。和其他的Set集合一样，ConcurrentSkipListSet是基于Map集合的，ConcurrentSkipListMap便是它的底层实现

    在多线程的环境下，ConcurrentSkipListSet中的contains、add、remove操作是安全的，多个线程可以安全地并发执行插入、移除和访问操作。但是对于批量操作 addAll、removeAll、retainAll 和 containsAll并不能保证以原子方式执行。理由很简单，因为addAll、removeAll、retainAll底层调用的还是contains、add、remove的方法，在批量操作时，只能保证每一次的contains、add、remove的操作是原子性的（即在进行contains、add、remove三个操作时，不会被其他线程打断），而不能保证每一次批量的操作都不会被其他线程打断。所以在进行批量操作时，需自行额外手动做一些同步、加锁措施，以此保证线程安全。另外，ConcurrentSkipListSet类不允许使用 null 元素，因为无法可靠地将 null 参数及返回值与不存在的元素区分开来。

- HashMap对应的ConcurrentHashMap

  * HashMap的并发安全版本是ConcurrentHashMap，但ConcurrentHashMap不允许 null 值。在大多数情况下，我们使用map都是读取操作，写操作比较少。因此ConcurrentHashMap针对读取操作做了大量的优化，所以ConcurrentHashMap具有很高的并发性，在高并发场景下表现良好

- TreeMap对应的ConcurrentSkipListMap

  * ConcurrentSkipListMap的底层是通过跳表来实现的。跳表是一个链表，但是通过使用“跳跃式”查找的方式使得插入、读取数据时复杂度变成了O(logn)。Tips：有人曾比较过ConcurrentHashMap和ConcurrentSkipListMap的性能，在4线程1.6万数据的条件下，ConcurrentHashMap 存取速度是ConcurrentSkipListMap 的4倍左右。

  * ConcurrentSkipListMap有几个ConcurrentHashMap不能比拟的优点

    - ConcurrentSkipListMap 的key是有序的

    - ConcurrentSkipListMap 支持更高的并发。ConcurrentSkipListMap的存取时间是O(logn)，和线程数几乎无关。也就是说在数据量一定的情况下，并发的线程越多，ConcurrentSkipListMap越能体现出其优势

    - 在非多线程的情况下，应当尽量使用TreeMap。此外对于并发性相对较低的并行程序可以使Collections.synchronizedSortedMap将TreeMap进行包装，也可以提供较好的效率。对于高并发程序，应当使用ConcurrentSkipListMap，能够提供更高的并发度。

      所以在多线程程序中，如果需要对Map的键值进行排序时，请尽量使用ConcurrentSkipListMap，可能得到更好的并发度。
      注意，调用ConcurrentSkipListMap的size时，由于多个线程可以同时对映射表进行操作，所以映射表需要遍历整个链表才能返回元素个数，这个操作是个O(log(n))的操作

### AQS

- AQS简介

  * Java并发包（JUC）中提供了很多并发工具，如ReentrangLock、Semaphore，它们的实现都用到了一个共同的基类--AbstractQueuedSynchronizer（抽象队列同步器），简称AQS

    AQS是JDK提供的一套用于实现基于FIFO等待队列的阻塞锁和相关的同步器的一个同步框架，它使用一个int类型的volatile变量（命名为state）来维护同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作。

    AbstractQueuedSynchronizer中对state的操作是原子的，且不能被继承。所有的同步机制的实现均依赖于对改变量的原子操作。为了实现不同的同步机制，我们需要创建一个非共有的（non-public internal）扩展了AQS类的内部辅助类来实现相应的同步逻辑。

    AbstractQueuedSynchronizer并不实现任何同步接口，它提供了一些可以被具体实现类直接调用的一些原子操作方法来重写相应的同步逻辑。AQS同时提供了独占模式（exclusive）和共享模式（shared）两种不同的同步逻辑。一般情况下，子类只需要根据需求实现其中一种模式，当然也有同时实现两种模式的同步类，如ReadWriteLock。

    使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的ReentrantLock，Semaphore，其他的诸如ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。

    同时，我们也能利用AQS非常轻松容易地构造出符合我们自己需求的同步器，由此可知AQS是Java并发包中最为核心的一个基类。

  * AbstractQueuedSynchronizer底层数据结构是一个双向链表，属于队列的一种实现

    * sync queue：同步队列，其中head节点主要负责后面的调度

    * Condition queue：单向链表，不是必须的，只有程序中使用到Condition的时候才会存在，可能会有多个Condition queue

<img src="C:\Users\hawk4\Desktop\临时\笔记\笔记图片\AbstractQueuedSynchronizer.jpg" alt="11" style="zoom:50%;" />

* 关于AQS里的state状态

  * AbstractQueuedSynchronizer维护了一个volatile int类型的变量，命名为state，用于表示当前同步状态。volatile虽然不能保证操作的原子性，但是保证了当前变量state的可见性
  
  * state的访问方式有三种：getState() setState() compareAndSetState()
  
    这三种操作均是原子操作，其中compareAndSetState的实现依赖于Unsafe的compareAndSwapInt()方法

* 资源自定义共享方式

  * AQS支持两种资源共享方式：Exclusive（独占，只有一个线程能执行，如ReentrantLock）和Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）。这样方便使用者实现不同类型的同步组件，独占式如ReentrantLock，共享式如Semaphore，CountDownLatch，组合式的如ReentrantReadWriteLock。总之，AQS为使用提供了底层支撑，如何组装实现，使用者可以自由发挥

* *同步器设计 Extra*

  同步器的设计是基于模板方法模式的，一般的使用方式是这样：

  - 使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）
  - 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。*这其实是模板方法模式的一个很经典的应用*。

  不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在底层实现好了。自定义同步器实现时主要实现以下几种方法

  ```java
  protected boolean isHeldExclusively()    // 该线程是否正在独占资源。只有用到condition才需要去实现它。
  protected boolean tryAcquire(int)        // 独占方式。尝试获取资源，成功则返回true，失败则返回false。
  protected boolean tryRelease(int)        // 独占方式。尝试释放资源，成功则返回true，失败则返回false。
  protected int tryAcquireShared(int)  // 共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
  protected boolean tryReleaseShared(int)  // 共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。
  ```

* *使用Extra*

  首先，我们需要去继承AbstractQueuedSynchronizer这个类，然后我们根据我们的需求去重写相应的方法，比如要实现一个独占锁，那就去重写tryAcquire，tryRelease方法，要实现共享锁，就去重写tryAcquireShared，tryReleaseShared；最后，在我们的组件中调用AQS中的模板方法就可以了，而这些模板方法是会调用到我们之前重写的那些方法的。也就是说，我们只需要很小的工作量就可以实现自己的同步组件，重写的那些方法，仅仅是一些简单的对于共享资源state的获取和释放操作，至于像是获取资源失败，线程需要阻塞之类的操作，自然是AQS帮我们完成了

* *具体实现Extra*

  * 首先AQS内部维护了一个CLH队列，来管理锁
  * 线程尝试获取锁，如果获取失败，则将等待信息等包装成一个Node结点，加入到同步队列Sync queue里
  * 不断重新尝试获取锁（当前结点为head的直接后继才会尝试），如果获取失败，则会阻塞自己，直到被唤醒
  * 当持有锁的线程释放锁的时候，会唤醒队列中的后继线程

*  *设计思想Extra*

  对于使用者来讲，我们无需关心获取资源失败，线程排队，线程阻塞/唤醒等一系列复杂的实现，这些都在AQS中为我们处理好了。我们只需要负责好自己的那个环节就好，也就是获取/释放共享资源state的姿势。很经典的模板方法设计模式的应用，AQS为我们定义好顶级逻辑的骨架，并提取出公用的线程入队列/出队列，阻塞/唤醒等一系列复杂逻辑的实现，将部分简单的可由使用者决定的操作逻辑延迟到子类中去实现即可

* 基于AQS的同步组件

  * CountDownLatch
  * Semaphore
  * CyclicBarrier
  * ReentrantLock
  * Condition
  * FutureTask

* 总结

  - 使用Node实现FIFO队列，可以用于构建锁或者其他同步装置的基础框架
  - 利用了一个int类型表示状态，有一个state的成员变量，表示获取锁的线程数（0没有线程获取锁，1有线程获取锁，大于1表示重入锁的数量），和一个同步组件ReentrantLock。状态信息通过procted级别的getState，setState，compareAndSetState进行操作
  - 使用方法是继承，然后复写AQS中的方法，基于模板方法模式
  - 子类通过继承并通过实现它的方法管理其状态{acquire和release}的方法操作状态
  - 可以同时实现排它锁和共享锁的模式（独占、共享）

#### CountDownLatch

* 概念 CountDownLatch是一个同步工具类，它允许一个或多个线程一直等待，直到其他线程执行完后再执行。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有框架服务之后执行。

* 实现方式 CountDownLatch是通过一个计数器来实现的，计数器的初始化值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就相应得减1。当计数器到达0时，表示所有的线程都已完成任务，然后在闭锁上等待的线程就可以恢复执行任务

![CountDownLatch](C:\Users\hawk4\Desktop\临时\笔记\笔记图片\countDownLatch.png)

* CountDownLatch的构造函数源码

```java
/**
 * Constructs a {@code CountDownLatch} initialized with the given count.
 *
 * @param count the number of times {@link #countDown} must be invoked
 *        before threads can pass through {@link #await}
 * @throws IllegalArgumentException if {@code count} is negative
 */
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```

计数器count是闭锁需要等待的线程数量，只能被设置一次，且CountDownLatch没有提供任何机制去重新设置计数器count。

与CountDownLatch的第一次交互是主线程等待其他线程。主线程必须在启动其他线程后立即调用CountDownLatch.await()方法。这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务。

其他N个线程必须引用CountDownLatch闭锁对象，因为它们需要通知CountDownLatch对象，它们各自完成了任务；这种通知机制是通过CountDownLatch.countDown()方法来完成的；每调用一次，count的值就减1，因此当N个线程都调用这个方法，count的值就等于0，然后主线程就可以通过await()方法，恢复执行自己的任务

* CountDownLatch使用场景
  * 实现最大的并行性：有时我们想同时启动多个线程，实现最大程度的并行性。例如，我们想测试一个单例类。如果我们创建一个初始计数器为1的CountDownLatch，并让其他所有线程都在这个锁上等待，只需要调用一次countDown()方法就可以让其他所有等待的线程同时恢复执行
  * 开始执行前等待N个线程完成各自任务：例如应用程序启动类要确保在处理用户请求前，所有N个外部系统都已经启动和运行了
  * 死锁检测：一个非常方便的使用场景是你用N个线程去访问共享资源，在每个测试阶段线程数量不同，并尝试产生死锁

#### Semaphore

* 主要作用

  可以控制同一时间并发执行的线程数。Semaphore有两个构造函数，参数permits表示许可数，它最后传递给了AQS的state值。线程在运行时首先获取许可，如果成功，许可数就减1，线程运行，当线程运行结束就释放许可，许可数就加1。如果许可数为0，则获取失败，线程位于AQS的等待队列中，它会被其它释放许可的线程唤醒。在创建Semaphore对象的时候还可以指定它的公平性。一般常用非公平的信号量，非公平信号量是指在获取许可时先尝试获取许可，而不必关心是否已有需要获取许可的线程位于等待队列中，如果获取失败，才会入列。而公平的信号量在获取许可时首先要查看等待队列中是否已有线程，如果有则入列

* 使用场景

  Semaphore可以用于做流量控制，特别公用资源有限的应用场景，比如数据库连接。假如有一个需求，要读取几万个文件的数据，因为都是IO密集型任务，我们可以启动几十个线程并发的读取，但是如果读到内存后，还需要存储到数据库中，而数据库的连接数只有10个，这时我们必须控制只有十个线程同时获取数据库连接保存数据，否则会报错无法获取数据库连接。这个时候，我们就可以使用Semaphore来做流控。

<img src="C:\Users\hawk4\Desktop\临时\笔记\笔记图片\Semaphore继承关系.png" alt="Semaphore继承关系" style="zoom: 40%;" />

* 常用方法

  ```java
  semaphore.Acquire()
  semaphore.tryAcquire()
  ```

  <img src="C:\Users\hawk4\Desktop\临时\笔记\笔记图片\tryAcquire方法.png" alt="tryAcquire方法" style="zoom:80%;" />

* 其他常用方法

  ```java
  int availablePermits()             // 返回此信号量中当前可用的许可证数。
  int getQueueLength()               // 返回正在等待获取许可证的线程数。
  boolean hasQueuedThreads()         // 是否有线程正在等待获取许可证。
  void reducePermits(int reduction)  // 减少reduction个许可证。是个protected方法。
  Collection getQueuedThreads()      // 返回所有等待获取许可证的线程集合。是个protected方法。
  ```

#### CyclicBarrier

* 概念

  字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。当某个线程调用了await方法之后，就会进入等待状态，并将计数器-1，直到所有线程调用await方法使计数器为0，才可以继续执行，由于计数器可以重复使用，所以我们又叫他循环屏障

  CyclicBarrier默认的构造方法是CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞

<img src="C:\Users\hawk4\Desktop\临时\笔记\笔记图片\CyclicBarrier.png" alt="CyclicBarrier" style="zoom:50%;" />

* CyclicBarrier的应用场景

  CyclicBarrier可以用于多线程计算数据，最后合并计算结果的应用场景。比如我们用一个Excel保存了用户所有银行流水，每个Sheet保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个sheet里的银行流水，都执行完之后，得到每个sheet的日均银行流水，最后，再用barrierAction用这些线程的计算结果，计算出整个Excel的日均银行流水

* CyclicBarrier和CountDownLatch的区别

  * CountDownLatch的计数器只能使用一次。而CyclicBarrier的计数器可以使用reset() 方法重置。所以CyclicBarrier能处理更为复杂的业务场景，比如如果计算发生错误，可以重置计数器，并让线程们重新执行一次。
  * CountDownLatch主要用于实现一个或n个线程需要等待其他线程完成某项操作之后，才能继续往下执行，描述的是一个或n个线程等待其他线程的关系，而CyclicBarrier是多个线程相互等待，知道满足条件以后再一起往下执行。描述的是多个线程相互等待的场景
  * CyclicBarrier还提供其他有用的方法，比如getNumberWaiting方法可以获得CyclicBarrier阻塞的线程数量。isBroken方法用来知道阻塞的线程是否被中断

* CyclicBarrier方法列表

<img src="C:\Users\hawk4\Desktop\临时\笔记\笔记图片\CyclicBarrier方法列表.png" alt="CyclicBarrier方法列表" style="zoom:80%;" />

* 基本使用

```java
public class CyclicBarrierExample1 {
    // 给定一个值，说明有多少个线程同步等待
    private static CyclicBarrier barrier = new CyclicBarrier(5);

    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool();

        for (int i = 0; i < 10; i++) {
            final int num = i;
            // 延迟1秒，方便观察
            Thread.sleep(1000);
            exec.execute(() -> {
                try {
                    CyclicBarrierExample1.race(num);
                } catch (Exception e) {
                    log.error("", e);
                }
            });
        }
        exec.shutdown();
    }

    private static void race(int num) throws Exception {
        Thread.sleep(1000);
        log.info("{} is ready", num);
        // 阻塞线程
        barrier.await();
        log.info("{} continue", num);
    }
}
```

以防await无限阻塞进程，我们可以设置await的超时时间，修改race方法

```java
private static void race(int num) throws Exception {
    Thread.sleep(1000);
    log.info("{} is ready", num);
    try {
        // 由于设置了超时时间后阻塞的线程可能会被中断，抛出BarrierException异常，如果想继续往下执行，需要加上try-catch
        barrier.await(2000, TimeUnit.MILLISECONDS);
    } catch (InterruptedException | TimeoutException | BrokenBarrierException e) {
        // isBroken方法用来知道阻塞的线程是否被中断
        log.warn("exception occurred {} {}. isBroken : {}", e.getClass().getName(), e.getMessage(), barrier.isBroken());
    }
    log.info("{} continue", num);
}
```

如果希望当所有线程到达屏障后就执行一个runnable的话，可以使用`CyclicBarrier(int parties, Runnable barrierAction)`构造函数传递一个runnable实例

```java
/**
 * 当线程全部到达屏障时，优先执行这里传入的runnable
 */
private static CyclicBarrier barrier = new CyclicBarrier(5, () -> log.info("callback is running"));
```

#### ReentrantLock

在Java里一共有两类锁，一类是synchornized同步锁，还有一种是JUC里提供的锁Lock，Lock是个接口，其核心实现类就是ReentrantLock

* synchornized与ReentrantLock的区别

| 对比维度                                                     | synchornized                                                 | ReentrantLock                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------- |
| 可重入性（线程进入锁的时候计数器就自增1，计数器下降为0则会释放锁） | 可重入                                                       | 可重入                         |
| 锁的实现                                                     | JVM实现，很难操作源码                                        | JDK实现，可以观察其源码        |
| 性能                                                         | 在引入偏向锁、轻量级锁/自旋锁后性能大大提升，官方建议无特殊要求时尽量使用synchornized，并且新版本的一些jdk源码都由之前的ReentrantLock改成了synchornized | 与优化后的synchornized相差不大 |
| 功能区别                                                     | 方便简洁，由编译器负责加锁和释放锁                           | 需手工操作锁的加锁和释放       |
| 锁粒度                                                       | 粗粒度，不灵活                                               | 细粒度，可灵活控制             |
| 可否指定公平锁                                               | 不可以                                                       | 可以                           |
| 可否放弃锁                                                   | 不可以                                                       | 可以                           |

* ReentrantLock实现：
  - 采用自旋锁，循环调用CAS操作来实现加锁，避免了使线程进入内核态的阻塞状态。想尽办法避免线程进入内核态的阻塞状态，是我们分析和理解锁设计的关键钥匙
* ReentrantLock独有的功能：
  - 可指定是公平锁还是非公平锁，所谓公平锁就是先等待的线程先获得锁
  - 提供了一个Condition类，可以分组唤醒需要唤醒的线程
  - 提供能够中断等待锁的线程的机制，`lock.lockInterruptibly()`

在ReentrantLock中，对于公平和非公平的定义是通过对同步器AQS的扩展加以实现的，也就是在tryAcquire的实现上做了语义的控制。

这里提到一个锁获取的公平性问题，如果在绝对时间上，先对锁进行获取的请求一定被先满足，那么这个锁是公平的，反之，是不公平的，也就是说等待时间最长的线程最有机会获取锁，也可以说锁的获取是有序的。ReentrantLock这个锁提供了一个构造函数，能够控制这个锁是否是公平的。

而锁的名字也是说明了这个锁具备了重复进入的可能，也就是说能够让当前线程多次的进行对锁的获取操作，这样的最大次数限制是`Integer.MAX_VALUE`，约21亿次左右。

事实上公平的锁机制往往没有非公平的效率高，因为公平的获取锁没有考虑到操作系统对线程的调度因素，这样造成JVM对于等待中的线程调度次序和操作系统对线程的调度之间的不匹配。对于锁的快速且重复的获取过程中，连续获取的概率是非常高的，而公平锁会压制这种情况，虽然公平性得以保障，但是响应比却下降了，但是并不是任何场景都是以TPS作为唯一指标的，因为公平锁能够减少“饥饿”发生的概率，等待越久的请求越是能够得到优先满足

在ReentrantLock 中，`lock()`方法是一个无条件的锁，与synchronize意思差不多，但是另一个方法 `tryLock()`方法只有在成功获取了锁的情况下才会返回true，如果别的线程当前正持有锁，则会立即返回false。如果为这个方法加上timeout参数，则会在等待timeout的时间才会返回false或者在获取到锁的时候返回true

* 其他常用方法

  ```java
  boolean isHeldByCurrentThread();   // 当前线程是否保持锁定
  boolean isLocked()  // 是否存在任意线程持有锁资源
  void lockInterruptbly()  // 如果当前线程未被中断，则获取锁定；如果已中断，则抛出异常(InterruptedException)
  int getHoldCount()   // 查询当前线程保持此锁定的个数，即调用lock()方法的次数
  int getQueueLength()   // 返回正等待获取此锁定的预估线程数
  int getWaitQueueLength(Condition condition)  // 返回与此锁定相关的约定condition的线程预估数
  boolean hasQueuedThread(Thread thread)  // 当前线程是否在等待获取锁资源
  boolean hasQueuedThreads()  // 是否有线程在等待获取锁资源
  boolean hasWaiters(Condition condition)  // 是否存在指定Condition的线程正在等待锁资源
  boolean isFair()   // 是否使用的是公平锁
  ```

#### Condition

Condition是一个多线程间协调通信的工具类，使得某个，或者某些线程一起等待某个条件（Condition），只有当该条件具备( signal 或者 signalAll方法被调用)时 ，这些等待线程才会被唤醒，从而重新争夺锁

```java
@Slf4j
public class LockExample6 {
    public static void main(String[] args) {
        // 构建ReentrantLock实例
        ReentrantLock reentrantLock = new ReentrantLock();
        // 从reentrantLock实例里获取condition实例
        Condition condition = reentrantLock.newCondition();

        // 线程1
        new Thread(() -> {
            try {
                // 线程1调用了lock方法，这时线程1就会加入到了AQS的等待队里面去
                reentrantLock.lock();
                log.info("wait signal"); // 1 等待信号
                // 调用await方法后，线程1就会从AQS队列里移除，这里其实就已经释放了锁，然后线程1会马上进入到condition队列里面去，等待一个信号
                condition.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info("get signal");  // 4 得到信号
            // 线程1释放锁，整个过程执行完毕
            reentrantLock.unlock();
        }).start();

        // 线程2
        new Thread(() -> {
            // 由于线程1中调用了await释放了锁的关系，所以线程2就会被唤醒获取到锁，加入到AQS等待队列中
            reentrantLock.lock();
            log.info("get lock");  // 2 获取锁
            try {
                // 睡眠3秒
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 调用signalAll发送信号的方法，此时condition等待队列里线程1所在的节点元素就会被取出，然后重新放到AQS等待队列里（注意此时线程1还没有被唤醒）
            condition.signalAll();
            log.info("send signal ~ ");   // 3 发送信号
            // 线程2释放锁，这时候AQS队列中只剩下线程1，然后AQS会按照从头到尾的顺序唤醒线程，于是线程1开始执行
            reentrantLock.unlock();
        }).start();
    }
}
```

#### ReentrantReadWriteLock
ReentrantReadWriteLock是Lock的另一种实现方式，我们已经知道了ReentrantLock是一个排他锁，同一时间只允许一个线程访问，而ReentrantReadWriteLock允许多个读线程同时访问，但不允许写线程和读线程、写线程和写线程同时访问。在没有任何读写锁的时候才能取得写入的锁，可用于实现悲观读取。相对于排他锁，提高了并发性。在实际应用中，大部分情况下对共享数据（如缓存）的访问都是读操作远多于写操作，这时ReentrantReadWriteLock能够提供比排他锁更好的并发性和吞吐量，所以读写锁适用于读多写少的情况。但读多写少的场景下可能会令写入线程遭遇饥饿，即写入线程迟迟无法获取到锁资源而处于等待状态。

与互斥锁相比，使用读写锁能否提升性能则取决于读写操作期间读取数据相对于修改数据的频率，以及数据的争用——即在同一时间试图对该数据执行读取或写入操作的线程数。

读写锁内部维护了两个锁，一个用于读操作，一个用于写操作。所有 ReadWriteLock实现都必须保证 writeLock操作的内存同步效果也要保持与相关 readLock的联系。也就是说，成功获取读锁的线程会看到写入锁之前版本所做的所有更新。
* ReentrantReadWriteLock支持功能
  * 非公平模式（默认）：连续竞争的非公平锁可能无限期地推迟一个或多个reader或writer线程，但吞吐量通常要高于公平锁
  * 公平模式：线程利用一个近似到达顺序的策略来争夺进入。当释放当前保持的锁时，可以为等待时间最长的单个writer线程分配写入锁，如果有一组等待时间大于所有正在等待的writer线程的reader，将为该组分配读者锁。试图获得公平写入锁的非重入的线程将会阻塞，除非读取锁和写入锁都自由（这意味着没有等待线程）
  * 支持可重入。读线程在获取了读锁后还可以获取读锁；写线程在获取了写锁之后既可以再次获取写锁又可以获取读锁
  * 还允许从写入锁降级为读取锁，其实现方式是：先获取写入锁，然后获取读取锁，最后释放写入锁。但是，从读取锁升级到写入锁是不允许的
  * 读取锁和写入锁都支持锁获取期间的中断
  * Condition支持。仅写入锁提供了一个 Conditon 实现；读取锁不支持 Conditon ，readLock().newCondition() 会抛出 UnsupportedOperationException
  * 监测：此类支持一些确定是读取锁还是写入锁的方法。这些方法设计用于监视系统状态，而不是同步控制

#### StampedLock

StampedLock是Java8引入的一种新的锁机制，简单的理解，可以认为它是读写锁的一个改进版本，读写锁虽然分离了读和写的功能，使得读与读之间可以完全并发，但是读和写之间依然是冲突的，读锁会完全阻塞写锁，它使用的依然是悲观的锁策略。如果有大量的读线程，它也有可能引起写线程的饥饿。而StampedLock则提供了一种乐观的读策略，这种乐观策略的锁非常类似于无锁的操作，使得乐观锁完全不会阻塞写线程。

StampedLock控制锁有三种形式，分别是写，读，和乐观读，重点在乐观锁。一个StampedLock，状态是由版本和模式两个部分组成。锁获取的方法返回的是一个数字作为票据（Stamp），他用相应的锁状态来表示并控制相关的访问，数字0表示没有写锁被授权访问，在读锁上分为悲观读和乐观读。

所谓的乐观读模式，也就是若读的操作很多，写的操作很少的情况下，你可以乐观地认为，写入与读取同时发生几率很少，因此不悲观地使用完全的读取锁定，程序可以查看读取资料之后，是否遭到写入执行的变更，再采取后续的措施（重新读取变更信息，或者抛出异常） ，这一个小小改进，可大幅度提高程序的吞吐量

* 适用场景

  乐观读取模式仅用于短时间读取操作时经常能够降低竞争和提高吞吐量。当然，它的使用在本质上是脆弱的。乐观读取的区域应该只包括字段，并且在validation之后用局部变量持有它们从而在后续使用。乐观模式下读取的字段值很可能是非常不一致的，所以它应该只用于那些你熟悉如何展示数据，从而你可以不断检查一致性和调用方法validate

* 优化点

  * 乐观读不阻塞悲观读和写操作，有利于获得写锁
  * 队列头结点采用有限次数SPINS次自旋（增加开销），增加获得锁几率（因为闯入的线程会竞争锁），有效够降低上下文切换
  * 读模式的集合通过一个公共节点被聚集在一起（cowait链），当队列尾节点为RMODE,通过CAS方法将该节点node添加至尾节点的cowait链中，node成为cowait中的顶元素，cowait构成了一个LIFO队列。
  * 不支持锁重入，如果只悲观读锁和写锁，效率没有ReentrantReadWriteLock高。

* 案例

```java
class Point {
    private double x, y;
    private final StampedLock sl = new StampedLock();

    void move(double deltaX, double deltaY) { // an exclusively locked method
        long stamp = sl.writeLock();
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            sl.unlockWrite(stamp);
        }
    }

    // 乐观读锁案例
    double distanceFromOrigin() { // A read-only method
        long stamp = sl.tryOptimisticRead(); //获得一个乐观读锁
        double currentX = x, currentY = y;  //将两个字段读入本地局部变量
        if (!sl.validate(stamp)) { //检查发出乐观读锁后同时是否有其他写锁发生？
            stamp = sl.readLock();  //如果没有，我们再次获得一个读悲观锁
            try {
                currentX = x; // 将两个字段读入本地局部变量
                currentY = y; // 将两个字段读入本地局部变量
            } finally {
                sl.unlockRead(stamp);
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }

    // 悲观读锁案例
    void moveIfAtOrigin(double newX, double newY) { // upgrade
        // Could instead start with optimistic, not read mode
        long stamp = sl.readLock();
        try {
            while (x == 0.0 && y == 0.0) { //循环，检查当前状态是否符合
                long ws = sl.tryConvertToWriteLock(stamp); //将读锁转为写锁
                if (ws != 0L) { //这是确认转为写锁是否成功
                    stamp = ws; //如果成功 替换票据
                    x = newX; //进行状态改变
                    y = newY;  //进行状态改变
                    break;
                } else { //如果不能成功转换为写锁
                    sl.unlockRead(stamp);  //我们显式释放读锁
                    stamp = sl.writeLock();  //显式直接进行写锁 然后再通过循环再试
                }
            }
        } finally {
            sl.unlock(stamp); //释放读锁或写锁
        }
    }
}
```

StampedLock 对吞吐量有巨大的改进，特别是在读线程越来越多的场景下。但StampedLock有一个复杂的API，对于加锁操作，很容易误用其他方法。StampedLock 可以说是Lock的一个很好的补充，吞吐量以及性能上的提升足以打动很多人了，但并不是说要替代之前Lock的东西，毕竟它还是有些应用场景的，起码API比StampedLock容易入手

#### 小结

* synchronized：JVM实现，不但可以通过一些监控工具监控，而且在出现未知异常的时候JVM也会自动帮我们释放锁
* ReentrantLock、ReentrantRead/WriteLock、StempedLock 他们都是对象层面的锁定，要想保证锁一定被释放，要放到finally里面，才会更安全一些。StempedLock对性能有很大的改进，特别是在读线程越来越多的情况下
* 使用
  * 在只有少量竞争者的时候，synchronized是一个很好的锁的实现
  * 竞争者不少，但是增长量是可以预估的，ReentrantLock是一个很好的锁的通用实现（适合使用场景的才是最好的，不是越高级越好）

#### JUC组件拓展

##### FutureTask

* 概述

  在Java中一般通过继承Thread类或者实现Runnable接口这两种方式来创建线程，但是这两种方式都有个缺陷，就是不能在执行完成后获取执行的结果，因此Java 1.5之后提供了Callable和Future接口，通过它们就可以在任务执行完毕之后得到任务的执行结果。

  而FutureTask则是J.U.C中的类，但不是AQS的子类，FutureTask是一个可删除的异步计算类。这个类提供了Future接口的的基本实现，使用相关方法启动和取消计算，查询计算是否完成，并检索计算结果。只有在计算完成时才能使用get方法检索结果;如果计算尚未完成，get方法将会阻塞。一旦计算完成，计算就不能重新启动或取消(除非使用runAndReset方法调用计算)

* Runnable与Callable以及Future接口对比
  * Runnable是一个接口，在它里面只声明了一个run()方法。由于run()方法返回值为void类型，所以在执行完任务之后无法返回任何结果
  * Callable接口也只声明了一个方法，这个方法叫做`call()`。可以看到Callable是个泛型接口，泛型V就是要`call()`方法返回的类型。Callable接口和Runnable接口很像，都可以被另外一个线程执行，但是正如前面所说的，Runnable不会返回数据也不能抛出异常。
  * Future也是一个接口，Future接口代表异步计算的结果，通过Future接口提供的方法可以查看异步计算是否执行完成，或者等待执行结果并获取执行结果，同时还可以取消执行。说白了Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成以及获取执行结果。其中执行结果通过get方法获取，该方法会阻塞直到任务返回结果
* Future接口的定义
  * cancel()方法用来取消异步任务的执行。如果异步任务已经完成或者已经被取消，或者由于某些原因不能取消，则会返回false。如果任务还没有被执行，则会返回true并且异步任务不会被执行。如果任务已经开始执行了但是还没有执行完成，若mayInterruptIfRunning为true，则会立即中断执行任务的线程并返回true，若mayInterruptIfRunning为false，则会返回true且不会中断任务执行线程。
  * isCanceled()方法用于判断任务是否被取消，如果任务在结束(正常执行结束或者执行异常结束)前被取消则返回true，否则返回false。
  * isDone()方法用于判断任务是否已经完成，如果完成则返回true，否则返回false。需要注意的是：任务执行过程中发生异常、任务被取消也属于任务已完成，也会返回true。
  * get()方法用于获取任务执行结果，如果任务还没完成则会阻塞等待直到任务执行完成。如果任务被取消则会抛出CancellationException异常，如果任务执行过程发生异常则会抛出ExecutionException异常，如果阻塞等待过程中被中断则会抛出InterruptedException异常。
  * get(long timeout,Timeunit unit)是带超时时间的get()版本，如果阻塞等待过程中超时则会抛出TimeoutException异常。

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

* Future主要提供了三种功能

  * 判断任务是否完成
* 能够中断任务
  * 能够获取任务执行结果

因为Future只是一个接口，所以是无法直接用来创建对象使用的，因此就有了下面的FutureTask。FutureTask的父类是RunnableFuture，而RunnableFuture则继承了Runnable和Future这两个接口。所以由此可知，FutureTask最终也属于是Callable类型的任务。如果往FutureTask的构造函数传入Runnable的话，也会被转换成Callable类型

* FutureTask继承图

![FutureTask继承图](C:\Users\hawk4\Desktop\临时\笔记\笔记图片\FutureTask继承图.png)

可以看到，FutureTask实现了RunnableFuture接口，则RunnableFuture接口继承了Runnable接口和Future接口，所以FutureTask既能当做一个Runnable直接被Thread执行，也能作为Future用来得到Callable的计算结果

* 使用场景

假设有一个很费时的逻辑需要计算，并且需要返回计算的结果，但这个结果又不是马上需要的。那么这时就可以使用FutureTask，用另外一个线程去进行计算，而当前线程在得到这个计算结果之前，就可以去执行其他的操作，等到需要这个结果时再通过Future得到即可。

FutureTask有两个构造器，支持传入Callable和Runnable类型，在使用 Runnable 时，需要多指定一个返回结果类型

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}

public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

##### ForkJoin

Fork/Join框架是Java7提供了的一个用于并行执行任务的框架， 是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架，其思想和map-reduce非常类似。

我们再通过Fork和Join这两个单词来理解下Fork/Join框架，Fork就是把一个大任务切分为若干子任务并行的执行，Join就是合并这些子任务的执行结果，最后得到这个大任务的结果。比如计算1+2+。。＋10000，可以分割成10个子任务，每个子任务分别对1000个数进行求和，最终汇总这10个子任务的结果

- Fork/Join的运行流程图

<img src="C:\Users\hawk4\Desktop\临时\笔记\笔记图片\ForkJoin的运行流程图.png" alt="ForkJoin的运行流程图" style="zoom: 67%;" />

* 工作窃取算法

  Fork/Join框架主要采用的是工作窃取（work-stealing）算法，该算法是指某个线程从其他队列里窃取任务来执行。

  那么为什么需要使用工作窃取算法呢？假如我们需要做一个比较大的任务，我们可以把这个任务分割为若干互不依赖的子任务，为了减少线程间的竞争，于是把这些子任务分别放到不同的队列里，并为每个队列创建一个单独的线程来执行队列里的任务，线程和队列一一对应，比如A线程负责处理A队列里的任务。但是有的线程会先把自己队列里的任务干完，而其他线程对应的队列里还有任务等待处理。干完活的线程与其等着，不如去帮其他线程干活，于是它就去其他线程的队列里窃取一个任务来执行。而在这时它们会访问同一个队列，所以为了减少窃取任务线程和被窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行。

  工作窃取算法的优点是充分利用线程进行并行计算，并减少了线程间的竞争，其缺点是在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且消耗了更多的系统资源，比如创建多个线程和多个双端队列。

  所以对于Fork/Join框架而言，当一个任务正在等待它使用join操作创建的子任务的结束时，执行这个任务的线程（工作线程）查找其他未被执行的任务并开始它的执行。通过这种方式，线程充分利用它们的运行时间，从而提高了应用程序的性能。

  * 运行流程图

<img src="C:\Users\hawk4\Desktop\临时\笔记\笔记图片\工作窃取运行流程图.png" alt="工作窃取运行流程图" style="zoom: 67%;" />

* Fork/Join框架执行的任务有以下局限性
  * 任务只能使用`fork()`和`join()`操作，作为同步机制。如果使用其他同步机制，工作线程不能执行其他任务，当它们在同步操作时。比如，在Fork/Join框架中，你使任务进入睡眠，那么在这睡眠期间内，正在执行这个任务的工作线程将不会执行其他任务。
  * 任务不应该执行I/O操作，如读或写数据文件。
  * 任务不能抛出检查异常，它必须包括必要的代码来处理它们。
* Fork/Join框架的核心主要是以下两个类
  * ForkJoinPool：它实现ExecutorService接口和work-stealing算法。它管理工作线程和提供关于任务的状态和它们执行的信息。
  * ForkJoinTask： 它是将在ForkJoinPool中执行的任务的基类。它提供在任务中执行`fork()`和`join()`操作的机制，并且这两个方法控制任务的状态。通常， 为了实现你的Fork/Join任务，你将实现两个子类的子类的类：RecursiveAction对于没有返回结果的任务和RecursiveTask 对于返回结果的任务。
* 使用示例

```java
package org.zero.concurrency.demo.example.aqs;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.Future;
import java.util.concurrent.RecursiveTask;

/**
 * @program: concurrency-demo
 * @description: ForkJoin 使用示例
 * @author: 01
 * @create: 2018-10-19 20:12
 **/
@Slf4j
public class ForkJoinTaskExample extends RecursiveTask<Integer> {
    private static final int THRESHOLD = 2;
    private int start;
    private int end;

    private ForkJoinTaskExample(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int sum = 0;

        //如果任务足够小就直接计算任务
        boolean canCompute = (end - start) <= THRESHOLD;
        if (canCompute) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else {
            // 如果任务大于阈值，就分裂成两个子任务计算
            int middle = (start + end) / 2;
            ForkJoinTaskExample leftTask = new ForkJoinTaskExample(start, middle);
            ForkJoinTaskExample rightTask = new ForkJoinTaskExample(middle + 1, end);

            // 执行子任务
            leftTask.fork();
            rightTask.fork();

            // 等待任务执行结束合并其结果
            int leftResult = leftTask.join();
            int rightResult = rightTask.join();

            // 合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }

    public static void main(String[] args) {
        ForkJoinPool forkjoinPool = new ForkJoinPool();

        //生成一个计算任务，计算1+2+3+4...+100
        ForkJoinTaskExample task = new ForkJoinTaskExample(1, 100);

        //执行一个任务
        Future<Integer> result = forkjoinPool.submit(task);

        try {
            log.info("result:{}", result.get());
        } catch (Exception e) {
            log.error("exception", e);
        }
    }
}
```

##### BlockingQueue

* 在新增的Concurrent包中，BlockingQueue很好的解决了多线程中，如何高效安全“传输”数据的问题，从名字也可以知道它是线程安全的。通过这些高效并且线程安全的队列类，为快速搭建高质量的多线程程序带来极大的便利。BlockingQueue 当获取队列元素但是队列为空时，会阻塞等待队列中有元素再返回；也支持添加元素时，如果队列已满，那么等到队列可以放入新元素时再放入。所以 BlockingQueue 主要应用于生产者消费者场景
* BlockingQueue 是一个接口，继承自 Queue，所以其实现类也可以作为 Queue 的实现来使用，而 Queue 又继承自 Collection 接口

| -       | Throws exception | Special value | Blocks         | Times out            |
| :------ | :--------------- | :------------ | :------------- | :------------------- |
| Insert  | add(e)           | offer(e)      | put(e)         | offer(e, time, unit) |
| Insert  | remove()         | poll()        | take()         | poll(time, unit)     |
| Examine | element()        | peek()        | not applicable | not applicable       |

* 说明
  * Throws Exceptions ：如果不能立即执行就抛出异常
  * Special Value：如果不能立即执行就返回一个特殊的值（null 或 true/false，取决于具体的操作
  * Blocks：如果不能立即执行就阻塞等待此操作，直到这个操作成功
  * Times Out：如果不能立即执行就阻塞一段时间，直到成功或者超时指定时间
* BlockingQueue 的实现类
  * ArrayBlockingQueue：它是一个有界的阻塞队列，内部实现是数组，需在初始化时指定容量大小，一旦指定大小就不能再变。采用FIFO方式存储元素
  * DelayQueue：阻塞内部元素，DelayQueue内部元素必须实现Delayed接口，Delayed接口又继承了Comparable接口，原因在于DelayQueue内部元素需要排序，一般情况下按元素过期时间优先级排序，DelayQueue内部采用PriorityQueue与ReentrantLock实现
  * LinkedBlockingQueue：使用独占锁实现的阻塞队列，大小配置可选，如果初始化时指定了大小，那么它就是有边界的。不指定就无边界（最大整型值）。内部实现是链表，采用FIFO形式保存数据。
  * PriorityBlockingQueue：带优先级的阻塞队列，无边界队列，允许插入null。插入的对象必须实现Comparator接口，队列优先级的排序规则就是按照我们对Comparable接口的实现来指定的。我们可以从PriorityBlockingQueue中获取一个迭代器，但这个迭代器并不保证能按照优先级的顺序进行迭代
  * SynchronousQueue：同步阻塞队列，只能插入一个元素，×××非缓存队列，不存储元素。其内部并没有数据缓存空间，你不能调用peek()方法来看队列中是否有数据元素，当然遍历这个队列的操作也是不允许的

### 线程池

#### 使用线程池的目的

* 线程是稀缺资源，不能频繁的创建。应当将其放入一个池子中，可以给其他任务进行复用，减少对象创建、消亡的开销，性能好
* 解耦作用；线程的创建于执行完全分开，方便维护
* 线程池可有效控制最大并发线程数，提高系统资源利用率，同时可以避免过多资源竞争，避免阻塞
* 线程池可提供定时执行、定期执行、单线程以及并发数控制等功能

#### 直接new Thread的弊端

* 每次new Thread 新建对象，性能差
* 线程缺乏统一管理，可能无限制的新建线程，相互竞争，常常会出现占用过多的系统资源导致死机或者发生OOM（out of memory 内存溢出），这种问题的原因不是因为单纯的new一个Thread，而是可能因为程序的bug或者设计上的缺陷导致不断new Thread造成的。
* 缺少更多功能，如更多执行、定期执行、线程中断等

#### threadPool类图

<img src="C:\Users\hawk4\Desktop\临时\笔记\笔记图片\threadPool类图.png" alt="threadPool类图" style="zoom: 80%;" />

在类图中，我们最常使用的是ThreadPoolExecutor和Executors，这两个类都可以创建线程池，其中ThreadPoolExecutor是可定制化的去创建线程池，而Executors则属于是工具类，该类中已经封装好了一些创建线程池的方法，直接调用相应的方法即可创建线程

![阿里关于Executor的规范](C:\Users\hawk4\Desktop\临时\笔记\笔记图片\阿里关于Executor的规范.png)

线程池体系里最为核心的类是ThreadPoolExecutor，也是功能最强的，ThreadPoolExecutor共有四个构造函数

![ThreadPoolExecutor构造函数](C:\Users\hawk4\Desktop\临时\笔记\笔记图片\ThreadPoolExecutor构造函数.png)

#### 线程池参数

* corePoolSize：核心线程数量
* maximumPoolSize：线程最大线程数
* workQueue：阻塞队列，存储等待执行的任务，很重要，会对线程池运行过程产生重大影响
* keepAliveTime：线程没有任务执行时最多保持多久时间终止（当线程中的线程数量大于corePoolSize的时候，如果这时没有新的任务提交核心线程外的线程不会立即销毁，而是等待，直到等待的时间超过keepAliveTime）
* unit：keepAliveTime的时间单位
* threadFactory：线程工厂，用来创建线程，若不设置则使用默认的工厂来创建线程，这样新创建出来的线程会具有相同的优先级，并且是非守护的线程，同时也会设置好名称
* rejectHandler：当拒绝处理任务时(阻塞队列满)的策略（AbortPolicy默认策略直接抛出异常、CallerRunsPolicy用调用者所在的线程执行任务、DiscardOldestPolicy丢弃队列中最靠前的任务并执行

corePoolSize、maximumPoolSize、workQueue 这三个参数的关系

* 如果运行的线程数量小于corePoolSize的时候，直接创建新线程来处理任务。即使线程池中的其他线程是空闲的。如果线程池中的线程数量大于corePoolSize且小于maximumPoolSize时，那么只有当workQueue满的时候才创建新的线程去处理任务。如果corePoolSize与maximumPoolSize是相同的，那么创建的线程池大小是固定的。这时如果有新任务提交，且workQueue未满时，就把请求放入workQueue中，等待空闲线程从workQueue取出任务进行处理。如果需要运行的线程数量大于maximumPoolSize时，并且此时workQueue也满了，那么就使用rejectHandler参数所指定的拒绝策略去进行处理

拒绝策略的实现类都在ThreadPoolExecutor中

![ThreadPoolExecutor拒绝策略类](C:\Users\hawk4\Desktop\临时\笔记\笔记图片\ThreadPoolExecutor拒绝策略类.png)

workQueue是保存待执行任务的一个阻塞队列，当我们提交一个新的任务到线程池后，线程池会根据当前池中正在运行的线程数量来决定该任务的处理方式，有三种处理方式

* SynchronusQueue
* LinkedBlockingQueue，若使用该队列，那么线程池中能够创建的最大线程数为corePoolSize，这时maximumPoolSize就不会起作用了。当线程池中所有的核心线程都是运行状态的时候，新的任务提交就会放入等待队列中。
* ArrayBlockingQueue，使用该队列可以将线程池中的最大线程数量限制为maximumPoolSize参数所指定的值，这种方式能够降低资源消耗，但是这种方式使得线程池对线程调度变的更困难。因为此时线程池与队列容量都是有限的了，所以想让线程池处理任务的吞吐率达到一个合理的范围，又想使我们的线程调度相对简单，并且还尽可能降低线程池对资源的消耗，那么我们就需要合理的设置corePoolSize和maximumPoolSize这两个参数的值

分配技巧： 如果想降低资源的消耗包括降低cpu使用率、操作系统资源的消耗、上下文切换的开销等等，可以设置一个较大的队列容量和较小的线程池容量，这样会降低线程池处理任务的吞吐量。如果我们提交的任务经常发生阻塞，我们可以考虑调用相关方法调整maximumPoolSize参数的值。如果我们的队列容量较小，通常需要把线程池的容量设置得大一些，这样cpu的使用率相对来说会高一些。但是如果线程池的容量设置的过大，提高任务的数量过多的时候，并发量会增加，那么线程之间的调度就是一个需要考虑的问题，这样反而可能会降低处理任务的吞吐量

#### 线程池状态

线程池有五种状态

<img src="C:\Users\hawk4\Desktop\临时\笔记\笔记图片\线程池状态.png" alt="线程池状态"  />

* running：运行状态，能接受新提交的任务，也能处理阻塞队列中的任务
* shutdown：关闭状态，不能处理新的任务，但却可以继续处理阻塞队列中已保存的任务。在线程池处于 RUNNING 状态时，调用 shutdown()方法会使线程池进入到该状态。（finalize() 方法在执行过程中也会调用shutdown()方法进入该状态）；
* stop：停止状态，不能接受新任务，也不处理队列中的任务，会中断正在处理任务的线程。在线程池处于 RUNNING 或 SHUTDOWN 状态时，调用 shutdownNow() 方法会使线程池进入到该状态；
* tidying：如果所有的任务都已终止了，workerCount (有效线程数) 为0，线程池进入该状态后会调用 terminated() 方法进入TERMINATED 状态。
* terminated：最终状态，在terminated() 方法执行完后进入该状态，默认terminated()方法中什么也没有做。

线程池常用方法

| 方法名                 | 描述                                      |
| :--------------------- | :---------------------------------------- |
| execute()              | 提交任务，交给线程池执行                  |
| submit()               | 提交任务，能够返回执行结果 execute+Future |
| shutdown()             | 关闭线程池，等待任务都执行完              |
| shutdownNow()          | 立刻关闭线程池，不等待任务执行完          |
| getTaskCount()         | 线程池已执行和未执行的任务总数            |
| getCompleteTaskCount() | 已完成的任务数量                          |
| getPoolSize()          | 线程池当前的线程数量                      |
| getActiveCount()       | 当前线程池中正在执行任务的线程数量        |

使用Executors创建线程池

| 方法名                  | 描述                                                         |
| :---------------------- | :----------------------------------------------------------- |
| newCachedThreadPool     | 创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程 |
| newFixedThreadPool      | 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待 |
| newScheduledThreadPool  | 创建一个定长线程池，支持定时及周期性任务执行                 |
| newSingleThreadExecutor | 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行 |

#### 使用ThreadPoolExecutor创建线程池

```java
package org.zero.concurrency.demo.example.threadpool;

import lombok.extern.slf4j.Slf4j;
import org.springframework.lang.NonNull;

import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @program: concurrency-demo
 * @description: ThreadPoolExecutor使用示例
 * @author: 01
 * @create: 2018-10-20 16:35
 **/
@Slf4j
public class ThreadPoolExample6 {
    public static void main(String[] args) {
        // 使用ArrayBlockingQueue作为其等待队列
        BlockingQueue<Runnable> blockingQueue = new ArrayBlockingQueue<>(5);
        // 使用自定义的ThreadFactory，目的是设置有意义的的线程名字，方便出错时回溯
        ThreadFactory namedThreadFactory = new MyThreadFactory("test-thread");

        // 创建线程池
        ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(
                3, 5, 1, TimeUnit.MINUTES, blockingQueue, namedThreadFactory,
                new ThreadPoolExecutor.AbortPolicy());

        // 执行任务
        poolExecutor.execute(() -> log.info("thread run"));

        // 关闭线程池
        poolExecutor.shutdown();
    }

    private static class MyThreadFactory implements ThreadFactory {
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final String namePrefix;

        private MyThreadFactory(String namePrefix) {
            this.namePrefix = namePrefix + "-";
        }

        @Override
        public Thread newThread(@NonNull Runnable r) {
            Thread t = new Thread(r, namePrefix + threadNumber.getAndIncrement());
            if (t.isDaemon()) {
                t.setDaemon(true);
            }
            if (t.getPriority() != Thread.NORM_PRIORITY) {
                t.setPriority(Thread.NORM_PRIORITY);
            }
            return t;
        }
    }
}
```

#### 线程池的合理配置

* CPU密集型任务，就需要尽量压榨CPU，参考值可以设置为NCPU+1，即CPU核心数量+1
* IO密集型任务，参考值可以设置为2*NCPU，即CPU核心数量的2倍
* 当线程池内需要执行的任务很小，小到执行任务的时间和任务调度的时间很接近，这时若使用线程池反而会更慢

### 死锁

* 概念 当两个以上的运算单元，双方都在等待对方停止运行，以获取系统资源，但是没有一方提前退出时，就称为死锁
* 死锁的四个条件 当同时满足时发生
  * 互斥条件：进程对所分配到的资源不允许其他进程进行访问，若其他进程访问该资源，只能等待，直至占有该资源的进程使用完成后释放该资源
  * 请求和保持条件：进程获得一定的资源之后，又对其他资源发出请求，但是该资源可能被其他进程占有，此事请求阻塞，但又对自己获得的资源保持不放
  * 不可剥夺条件：是指进程已获得的资源，在未完成使用之前，不可被剥夺，只能在使用完后自己释放
  * 环路等待条件：是指进程发生死锁后，必然存在一个进程–资源之间的环形链
* 解决死锁的四种方式
  * 预防死锁：通过设置一些限制条件，去破坏产生死锁的必要条件
  * 避免死锁：在资源分配过程中，使用某种方法避免系统进入不安全的状态，从而避免发生死锁
  * 检测死锁：允许死锁的发生，但是通过系统的检测之后，采取一些措施，将死锁清除掉
  * 解除死锁：该方法与检测死锁配合使用

### Spring与线程安全

Spring作为一个IOC/DI容器，帮助我们管理了许许多多的“bean”。但其实，Spring并没有保证这些对象的线程安全，需要由开发者自己编写解决线程安全问题的代码。

* Spring bean

  Spring对每个bean提供了一个scope属性来表示该bean的作用域。它是bean的生命周期。例如，一个scope为singleton的bean，在第一次被注入时，会创建为一个单例对象，该对象会一直被复用到应用结束。* * 

  * singleton：默认的scope，每个scope为singleton的bean都会被定义为一个单例对象，该对象的生命周期是与Spring IOC容器一致的（但在第一次被注入时才会创建）。在整个Spring IoC容器里，只有一个bean实例，所有线程共享该实例。
  * prototype：bean被定义为在每次注入时都会创建一个新的对象。每次请求都会创建并返回一个新的实例，所有线程都有单独的实例使用，这种方式是比较安全的，但会消耗大量内存和计算资源。
  * request（请求范围实例）：bean被定义为在每个HTTP请求中创建一个单例对象，也就是说在单个请求中都会复用这一个单例对象。每当接受到一个HTTP请求时，就分配一个唯一实例，这个实例在整个请求周期都是唯一的。
  * session（会话范围实例）：bean被定义为在一个session的生命周期内创建一个单例对象。在每个用户会话周期内，分配一个实例，这个实例在整个会话周期都是唯一的，所有同一会话范围的请求都会共享该实例。
  * application：bean被定义为在ServletContext的生命周期中复用一个单例对象。
  * websocket：bean被定义为在websocket的生命周期中复用一个单例对象。
  * globalsession（全局会话范围实例）：这与会话范围实例大部分情况是一样的，只是在使用到portlet时，由于每个portlet都有自己的会话，如果一个页面中有多个portlet而需要共享一个bean时，才会用到

* 无状态的对象

  即是自身没有状态的对象，自然也就不会因为多个线程的交替调度而破坏自身状态导致线程安全问题。无状态对象包括我们经常使用的DO、DTO、VO这些只作为数据的实体模型的贫血对象，还有Service、DAO和Controller，这些对象并没有自己的状态，它们只是用来执行某些操作的。例如，每个DAO提供的函数都只是对数据库的CRUD，而且每个数据库Connection都作为函数的局部变量（局部变量是在用户栈中的，而且用户栈本身就是线程私有的内存区域，所以不存在线程安全问题），用完即关（或交还给连接池）。

* 线程安全的几种情况
  * 常量始终是线程安全的，因为只存在读操作。
  * 每次调用方法前都新建一个实例是线程安全的，因为不会访问共享的资源。
  * 局部变量是线程安全的。因为每执行一个方法，都会在独立的空间创建局部变量，它不是共享的资源。局部变量包括方法的参数变量和方法内变量

Spring没有对bean的多线程安全问题做出任何保证与措施。对于每个bean的线程安全问题，根本原因是每个bean自身的设计没有在bean中声明任何有状态的实例变量或类变量，如果必须如此，那么就使用ThreadLocal把变量变为线程私有的，如果bean的实例变量或类变量需要在多个线程之间共享，那么就只能使用synchronized、lock、CAS等这些实现线程同步的方法

#### Hashmap与ConcurrentHashMap

https://blog.51cto.com/zero01/2307070