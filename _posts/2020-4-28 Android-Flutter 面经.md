# 面经

# JAVA 篇

### GC机制

垃圾回收需要完成两件事：找到垃圾，回收垃圾。 找到垃圾一般的话有两种方法：

- **引用计数法：** 当一个对象被引用时，它的引用计数器会加一，垃圾回收时会清理掉引用计数为0的对象。但这种方法有一个问题，比方说有两个对象 A 和 B，A 引用了 B，B 又引用了 A，除此之外没有别的对象引用 A 和 B，那么 A 和 B 在我们看来已经是垃圾对象，需要被回收，但它们的引用计数不为 0，没有达到回收的条件。正因为这个循环引用的问题，Java 并没有采用引用计数法。
- **可达性分析法：** 我们把 Java 中对象引用的关系看做一张图，从根级对象不可达的对象会被垃圾收集器清除。根级对象一般包括 Java 虚拟机栈中的对象、本地方法栈中的对象、方法区中的静态对象和常量池中的常量。 回收垃圾的话有这么四种方法：
- **标记清除算法：** 顾名思义分为两步，标记和清除。首先标记到需要回收的垃圾对象，然后回收掉这些垃圾对象。标记清除算法的缺点是清除垃圾对象后会造成内存的碎片化。
- **复制算法：** 复制算法是将存活的对象复制到另一块内存区域中，并做相应的内存整理工作。复制算法的优点是可以避免内存碎片化，缺点也显而易见，它需要两倍的内存。
- **标记整理算法：** 标记整理算法也是分两步，先标记后整理。它会标记需要回收的垃圾对象，清除掉垃圾对象后会将存活的对象压缩，避免了内存的碎片化。
- **分代算法：** 分代算法将对象分为新生代和老年代对象。那么为什么做这样的区分呢？主要是在Java运行中会产生大量对象，这些对象的生命周期会有很大的不同，有的生命周期很长，有的甚至使用一次之后就不再使用。所以针对不同生命周期的对象采用不同的回收策略，这样可以提高GC的效率。

新生代对象分为三个区域：Eden 区和两个 Survivor 区。新创建的对象都放在 Eden区，当 Eden 区的内存达到阈值之后会触发 Minor GC，这时会将存活的对象复制到一个 Survivor 区中，这些存活对象的生命存活计数会加一。这时 Eden 区会闲置，当再一次达到阈值触发 Minor GC 时，会将Eden区和之前一个 Survivor 区中存活的对象复制到另一个 Survivor 区中，采用的是我之前提到的复制算法，同时它们的生命存活计数也会加一。

这个过程会持续很多遍，直到对象的存活计数达到一定的阈值后会触发一个叫做晋升的现象：新生代的这个对象会被放置到老年代中。 老年代中的对象都是经过多次 GC 依然存活的生命周期很长的 Java 对象。当老年代的内存达到阈值后会触发 Major GC，采用的是标记整理算法。

### JVM内存区域的划分，哪些区域会发生 OOM

JVM 的内存区域可以分为两类：线程私有和区域和线程共有的区域。 线程私有的区域：程序计数器、JVM 虚拟机栈、本地方法栈 线程共有的区域：堆、方法区、运行时常量池

- **程序计数器。** 每个线程有有一个私有的程序计数器，任何时间一个线程都只会有一个方法正在执行，也就是所谓的当前方法。程序计数器存放的就是这个当前方法的JVM指令地址。
- **JVM虚拟机栈。** 创建线程的时候会创建线程内的虚拟机栈，栈中存放着一个个的栈帧，对应着一个个方法的调用。JVM 虚拟机栈有两种操作，分别是压栈和出站。栈帧中存放着局部变量表、方法返回值和方法的正常或异常退出的定义等等。
- **本地方法栈。** 跟 JVM 虚拟机栈比较类似，只不过它支持的是 Native 方法。
- **堆。** 堆是内存管理的核心区域，用来存放对象实例。几乎所有创建的对象实例都会直接分配到堆上。所以堆也是垃圾回收的主要区域，垃圾收集器会对堆有着更细的划分，最常见的就是把堆划分为新生代和老年代。
- 方法区。方法区主要存放类的结构信息，比如静态属性和方法等等。
- 运行时常量池。运行时常量池位于方法区中，主要存放各种常量信息。

其实除了程序计数器，其他的部分都会发生 OOM。

- **堆。** 通常发生的 OOM 都会发生在堆中，最常见的可能导致 OOM 的原因就是内存泄漏。
- **JVM虚拟机栈和本地方法栈。** 当我们写一个递归方法，这个递归方法没有循环终止条件，最终会导致 StackOverflow 的错误。当然，如果栈空间扩展失败，也是会发生 OOM 的。
- 方法区。方法区现在基本上不太会发生 OOM，但在早期内存中加载的类信息过多的情况下也是会发生 OOM 的。

### java的内存模型

Java内存模型规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中是用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的工作内存和主存之间进行数据同步进行。

**原子性**

在Java中，为了保证原子性，提供了两个高级的字节码指令`monitorenter`和`monitorexit`。在synchronized的实现原理文章中，介绍过，这两个字节码，在Java中对应的关键字就是`synchronized`。

因此，在Java中可以使用`synchronized`来保证方法和代码块内的操作是原子性的。

**可见性**

Java内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值的这种依赖主内存作为传递媒介的方式来实现的。

Java中的`volatile`关键字提供了一个功能，那就是被其修饰的变量在被修改后可以立即同步到主内存，被其修饰的变量在每次是用之前都从主内存刷新。因此，可以使用`volatile`来保证多线程操作时变量的可见性。

除了`volatile`，Java中的`synchronized`和`final`两个关键字也可以实现可见性。只不过实现方式不同，这里不再展开了。

**有序性**

在Java中，可以使用`synchronized`和`volatile`来保证多线程之间操作的有序性。实现方式有所区别：

`volatile`关键字会禁止指令重排。`synchronized`关键字保证同一时刻只允许一条线程操作。

好了，这里简单的介绍完了Java并发编程中解决原子性、可见性以及有序性可以使用的关键字。读者可能发现了，好像`synchronized`关键字是万能的，他可以同时满足以上三种特性，这其实也是很多人滥用`synchronized`的原因。

但是`synchronized`是比较影响性能的，虽然编译器提供了很多锁优化技术，但是也不建议过度使用

### 类加载过程

Java 中类加载分为 3 个步骤：加载、链接、初始化。

- **加载。** 加载是将字节码数据从不同的数据源读取到JVM内存，并映射为 JVM 认可的数据结构，也就是 Class 对象的过程。数据源可以是 Jar 文件、Class 文件等等。如果数据的格式并不是 ClassFile 的结构，则会报 ClassFormatError。
- **链接。**链接是类加载的核心部分，这一步分为 3 个步骤：验证、准备、解析。
- **验证。** 验证是保证JVM安全的重要步骤。JVM需要校验字节信息是否符合规范，避免恶意信息和不规范数据危害JVM运行安全。如果验证出错，则会报VerifyError。
- **准备。** 这一步会创建静态变量，并为静态变量开辟内存空间。
- **解析。** 这一步会将符号引用替换为直接引用。
- **初始化。** 初始化会为静态变量赋值，并执行静态代码块中的逻辑。

### 双亲委派模型

类加载器大致分为3类：启动类加载器、扩展类加载器、应用程序类加载器。

- 启动类加载器主要加载 `jre/lib`下的`jar`文件。
- 扩展类加载器主要加载 `jre/lib/ext` 下的`jar`文件。
- 应用程序类加载器主要加载 `classpath` 下的文件。

所谓的双亲委派模型就是当加载一个类时，会优先使用父类加载器加载，当父类加载器无法加载时才会使用子类加载器去加载。这么做的目的是为了避免类的重复加载。

### HashMap 的原理

HashMap 的内部可以看做数组+链表的复合结构。数组被分为一个个的桶(bucket)。哈希值决定了键值对在数组中的寻址。具有相同哈希值的键值对会组成链表。需要注意的是当链表长度超过阈值(默认是8)的时候会触发树化，链表会变成树形结构。

**把握HashMap的原理需要关注4个方法：hash、put、get、resize。**

- **hash方法。** 将 key 的 hashCode 值的高位数据移位到低位进行异或运算。这么做的原因是有些 key 的 hashCode 值的差异集中在高位，而哈希寻址是忽略容量以上高位的，这种做法可以有效避免哈希冲突。
- **put 方法。**put 方法主要有以下几个步骤：
- 通过 hash 方法获取 hash 值，根据 hash 值寻址。
- 如果未发生碰撞，直接放到桶中。
- 如果发生碰撞，则以链表形式放在桶后。
- 当链表长度大于阈值后会触发树化，将链表转换为红黑树。
- 如果数组长度达到阈值，会调用 resize 方法扩展容量。
- **get方法。**get 方法主要有以下几个步骤：
- 通过 hash 方法获取 hash 值，根据 hash 值寻址。
- 如果与寻址到桶的 key 相等，直接返回对应的 value。
- 如果发生冲突，分两种情况。如果是树，则调用 getTreeNode 获取 value；如果是链表则通过循环遍历查找对应的 value。
- **resize 方法。**resize 做了两件事：
- 将原数组扩展为原来的 2 倍
- 重新计算 index 索引值，将原节点重新放到新的数组中。这一步可以将原先冲突的节点分散到新的桶中。

### sleep 和 wait 的区别

- sleep 方法是 Thread 类中的静态方法，wait 是 Object 类中的方法
- sleep 并不会释放同步锁，而 wait 会释放同步锁
- sleep 可以在任何地方使用，而 wait 只能在同步方法或者同步代码块中使用
- sleep 中必须传入时间，而 wait 可以传，也可以不传，不传时间的话只有 notify 或者 notifyAll 才能唤醒，传时间的话在时间之后会自动唤醒

### join 的用法

join 方法通常是保证线程间顺序调度的一个方法，它是 Thread 类中的方法。比方说在线程 A 中执行线程 `B.join()`，这时线程 A 会进入等待状态，直到线程 B 执行完毕之后才会唤醒，继续执行A线程中的后续方法。

join 方法可以传时间参数，也可以不传参数，不传参数实际上调用的是 `join(0)`。它的原理其实是使用了 wait 方法，join 的原理如下：

```java
public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }
复制代码
```

### volatile

一般提到 volatile，就不得不提到内存模型相关的概念。我们都知道，在程序运行中，每条指令都是由 CPU 执行的，而指令的执行过程中，势必涉及到数据的读取和写入。程序运行中的数据都存放在主存中，这样会有一个问题，由于 CPU 的执行速度是要远高于主存的读写速度，所以直接从主存中读写数据会降低 CPU 的效率。为了解决这个问题，就有了高速缓存的概念，在每个 CPU 中都有高速缓存，它会事先从主存中读取数据，在 CPU 运算之后在合适的时候刷新到主存中。

这样的运行模式在单线程中是没有任何问题的，但在多线程中，会导致缓存一致性的问题。举个简单的例子：`i=i+1` ,在两个线程中执行这句代码，假设i的初始值为0。我们期望两个线程运行后得到2，那么有这样的一种情况，两个线程都从主存中读取i到各自的高速缓存中，这时候两个线程中的i都为0。在线程1执行完毕得到`i=1`，将之刷新到主存后，线程2开始执行，由于线程2中的i是高速缓存中的0，所以在执行完线程2之后刷新到主存的i仍旧是1。

所以这就导致了对共享变量的缓存一致性的问题，那么为了解决这个问题，提出了缓存一致性协议：当 CPU 在写数据时，如果发现操作的是共享变量，它会通知其他 CPU 将它们内部的这个共享变量置为无效状态，当其他 CPU 读取缓存中的共享变量时，发现这个变量是无效的，它会从新从主存中读取最新的值。

**在Java的多线程开发中，有三个重要概念：原子性、可见性、有序性。**

- **原子性：**一个或多个操作要么都不执行，要么都执行。
- **可见性：** 一个线程中对共享变量(类中的成员变量或静态变量)的修改，在其他线程立即可见。
- **有序性：** 程序执行的顺序按照代码的顺序执行。 把一个变量声明为volatile，其实就是保证了可见性和有序性。 可见性我上面已经说过了，在多线程开发中是很有必要的。这个有序性还是得说一下，为了执行的效率，有时候会发生指令重排，这在单线程中指令重排之后的输出与我们的代码逻辑输出还是一致的。但在多线程中就可能发生问题，volatile在一定程度上可以避免指令重排。

volatile的原理是在生成的汇编代码中多了一个lock前缀指令，这个前缀指令相当于一个内存屏障，这个内存屏障有3个作用：

- 确保指令重排的时候不会把屏障后的指令排在屏障前，确保不会把屏障前的指令排在屏障后。
- 修改缓存中的共享变量后立即刷新到主存中。
- 当执行写操作时会导致其他CPU中的缓存无效。

### volatile和synchronize的区别

**volatile**

它所修饰的**变量**不保留拷贝，直接访问主内存中的。

在Java内存模型中，有main memory，每个线程也有自己的memory (例如寄存器)。为了性能，一个线程会在自己的memory中保持要访问的变量的副本。这样就会出现同一个变量在某个瞬间，在一个线程的memory中的值可能与另外一个线程memory中的值，或者main memory中的值不一致的情况。 一个变量声明为volatile，就意味着这个变量是随时会被其他线程修改的，因此不能将它cache在线程memory中。

使用场景

您只能在有限的一些情形下使用 volatile 变量替代锁。要使 volatile 变量提供理想的线程安全，必须同时满足下面两个条件：

1)对变量的写操作不依赖于当前值。

2)该变量没有包含在具有其他变量的不变式中。

volatile最适用一个线程写，多个线程读的场合。

如果有多个线程并发写操作，仍然需要使用锁或者线程安全的容器或者原子变量来代替。

**synchronized**

当它用来修饰**一个方法或者一个代码块**的时候，能够保证在同一时刻最多只有一个线程执行该段代码。

1. 当两个并发线程访问同一个对象object中的这个synchronized(this)同步代码块时，一个时间内只能有一个线程得到执行。另一个线程必须等待当前线程执行完这个代码块以后才能执行该代码块。
2. 然而，当一个线程访问object的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该object中的非synchronized(this)同步代码块。
3. 尤其关键的是，当一个线程访问object的一个synchronized(this)同步代码块时，其他线程对object中所有其它synchronized(this)同步代码块的访问将被阻塞。
4. 当一个线程访问object的一个synchronized(this)同步代码块时，它就获得了这个object的对象锁。结果，其它线程对该object对象所有同步代码部分的访问都被暂时阻塞。

**区别**

1. volatile是变量修饰符，而synchronized则作用于一段代码或方法。
2. volatile只是在线程内存和“主”内存间同步某个变量的值；而synchronized通过锁定和解锁某个监视器同步所有变量的值, 显然synchronized要比volatile消耗更多资源。
3. volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。
4. volatile保证数据的可见性，但**不能保证原子性**；而synchronized可以保证原子性，也可以间接保证可见性，因为它会将私有内存中和公共内存中的数据做同步。
5. volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化。

线程安全包含原子性和可见性两个方面，Java的同步机制都是围绕这两个方面来确保线程安全的。

关键字volatile主要使用的场合是在多个线程中可以感知实例变量被修改，并且可以获得最新的值使用，也就是多线程读取共享变量时可以获得最新值使用。

关键字volatile提示线程每次从共享内存中读取变量，而不是私有内存中读取，这样就保证了同步数据的可见性。但是要注意的是：如果修改实例变量中的数据

### ThreadLocal的作用

ThreadLocal的作用是提供线程内的局部变量，说白了，就是在各线程内部创建一个变量的副本，相比于使用各种锁机制访问变量，ThreadLocal的思想就是用空间换时间，使各线程都能访问属于自己这一份的变量副本，变量值不互相干扰，减少同一个线程内的多个函数或者组件之间一些公共变量传递的复杂度。

**get函数**用来获取与当前线程关联的ThreadLocal的值，如果当前线程没有该ThreadLocal的值，则调用**initialValue函数**获取初始值返回，initialValue是protected类型的，所以一般我们使用时需要继承该函数，给出初始值。而**set函数**是用来设置当前线程的该ThreadLocal的值，**remove函数**用来删除ThreadLocal绑定的值，在某些情况下需要手动调用，防止内存泄露。

### Java中生产者与消费者模式

生产者消费者模式要保证的是当缓冲区满的时候生产者不再生产对象，当缓冲区空时，消费者不再消费对象。实现机制就是当缓冲区满时让生产者处于等待状态，当缓冲区为空时让消费者处于等待状态。当生产者生产了一个对象后会唤醒消费者，当消费者消费一个对象后会唤醒生产者。

**三种种实现方式：wait 和 notify、await 和 signal、BlockingQueue。**

- wait 和 notify

```java
//wait和notify
import java.util.LinkedList;
public class StorageWithWaitAndNotify {
    private final int MAX_SIZE = 10;
    private LinkedList<Object> list = new LinkedList<Object>();
    public void produce() {
        synchronized (list) {
            while (list.size() == MAX_SIZE) {
                System.out.println("仓库已满：生产暂停");
                try {
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            list.add(new Object());
            System.out.println("生产了一个新产品，现库存为：" + list.size());
            list.notifyAll();
        }
    }
    public void consume() {
        synchronized (list) {
            while (list.size() == 0) {
                System.out.println("库存为0：消费暂停");
                try {
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            list.remove();
            System.out.println("消费了一个产品，现库存为：" + list.size());
            list.notifyAll();
        }
    }
}
```

- await 和 signal

```java
import java.util.LinkedList;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;
class StorageWithAwaitAndSignal {
    private final int                MAX_SIZE = 10;
    private       ReentrantLock      mLock    = new ReentrantLock();
    private       Condition          mEmpty   = mLock.newCondition();
    private       Condition          mFull    = mLock.newCondition();
    private       LinkedList<Object> mList    = new LinkedList<Object>();
    public void produce() {
        mLock.lock();
        while (mList.size() == MAX_SIZE) {
            System.out.println("缓冲区满，暂停生产");
            try {
                mFull.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        mList.add(new Object());
        System.out.println("生产了一个新产品，现容量为：" + mList.size());
        mEmpty.signalAll();
        mLock.unlock();
    }
    public void consume() {
        mLock.lock();
        while (mList.size() == 0) {
            System.out.println("缓冲区为空，暂停消费");
            try {
                mEmpty.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        mList.remove();
        System.out.println("消费了一个产品，现容量为：" + mList.size());
        mFull.signalAll();
        mLock.unlock();
    }
}

```

- BlockingQueue

```java
import java.util.concurrent.LinkedBlockingQueue;
public class StorageWithBlockingQueue {
    private final int                         MAX_SIZE = 10;
    private       LinkedBlockingQueue<Object> list     = new LinkedBlockingQueue<Object>(MAX_SIZE);
    public void produce() {
        if (list.size() == MAX_SIZE) {
            System.out.println("缓冲区已满，暂停生产");
        }
        try {
            list.put(new Object());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("生产了一个产品，现容量为：" + list.size());
    }
    public void consume() {
        if (list.size() == 0) {
            System.out.println("缓冲区为空，暂停消费");
        }
        try {
            list.take();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("消费了一个产品，现容量为：" + list.size());
    }
}
复制代码
```

### final、finally、finalize区别

final 可以修饰类、变量和方法。修饰类代表这个类不可被继承。修饰变量代表此变量不可被改变。修饰方法表示此方法不可被重写 (override)。

finally 是保证重点代码一定会执行的一种机制。通常是使用 try-finally 或者 try-catch-finally 来进行文件流的关闭等操作。

finalize 是 Object 类中的一个方法，它的设计目的是保证对象在垃圾收集前完成特定资源的回收。finalize 机制现在已经不推荐使用，并且在 JDK 9已经被标记为 deprecated。

### Java 中单例模式

Java 中常见的单例模式实现有这么几种：饿汉式、双重判断的懒汉式、静态内部类实现的单例、枚举实现的单例。 这里着重讲一下双重判断的懒汉式和静态内部类实现的单例。

双重判断的懒汉式：

```java
public class SingleTon {
    //需要注意的是volatile
    private static volatile SingleTon mInstance;
    private SingleTon() {
    }
    public static SingleTon getInstance() {
        if (mInstance == null) { 
            synchronized (SingleTon.class) {
                if (mInstance == null) {
                    mInstance=new SingleTon();
                }
            }
        }
        return mInstance;
    }
}
```

双重判断的懒汉式单例既满足了延迟初始化，又满足了线程安全。通过 synchronized 包裹代码来实现线程安全，通过双重判断来提高程序执行的效率。这里需要注意的是单例对象实例需要有 volatile 修饰，如果没有 volatile 修饰，在多线程情况下可能会出现问题。原因是这样的，`mInstance=new SingleTon()`这一句代码并不是一个原子操作，它包含三个操作：

1. 给 mInstance 分配内存
2. 调用 SingleTon 的构造方法初始化成员变量
3. 将 mInstance 指向分配的内存空间（在这一步 mInstance 已经不为 null 了）

我们知道 JVM 会发生指令重排，正常的执行顺序是`1-2-3`，但发生指令重排后可能会导致`1-3-2`。我们考虑这样一种情况，当线程 A 执行到`1-3-2`的3步骤暂停了，这时候线程 B 调用了 getInstance，走到了最外层的if判断上，由于最外层的 if 判断并没有 synchronized 包裹，所以可以执行到这一句，这时候由于线程 A 已经执行了步骤3，此时 mInstance 已经不为 null 了，所以线程B直接返回了 mInstance。但其实我们知道，完整的初始化必须走完这三个步骤，由于线程 A 只走了两个步骤，所以一定会报错的。

解决的办法就是使用 volatile 修饰 mInstance，我们知道 volatile 有两个作用：保证可见性和禁止指令重排，在这里关键在于禁止指令重排，禁止指令重排后保证了不会发生上述问题。

静态内部类实现的单例：

```java
class SingletonWithInnerClass {
    private SingletonWithInnerClass() {
    }
    private static class SingletonHolder{
        private static SingletonWithInnerClass INSTANCE=new SingletonWithInnerClass();
    }
    public SingletonWithInnerClass getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

由于外部类的加载并不会导致内部类立即加载，只有当调用 getInstance 的时候才会加载内部类，所以实现了延迟初始化。由于类只会被加载一次，并且类加载也是线程安全的，所以满足我们所有的需求。静态内部类实现的单例也是最为推荐的一种方式。

### Java中引用类型的区别，具体的使用场景

Java中引用类型分为四类：强引用、软引用、弱引用、虚引用。

- **强引用：** 强引用指的是通过 new 对象创建的引用，垃圾回收器即使是内存不足也不会回收强引用指向的对象。
- **软引用：** 软引用是通过 SoftRefrence 实现的，它的生命周期比强引用短，在内存不足，抛出 OOM 之前，垃圾回收器会回收软引用引用的对象。软引用常见的使用场景是存储一些内存敏感的缓存，当内存不足时会被回收。
- **弱引用：** 弱引用是通过 WeakRefrence 实现的，它的生命周期比软引用还短，GC 只要扫描到弱引用的对象就会回收。弱引用常见的使用场景也是存储一些内存敏感的缓存。
- **虚引用：** 虚引用是通过 FanttomRefrence 实现的，它的生命周期最短，随时可能被回收。如果一个对象只被虚引用引用，我们无法通过虚引用来访问这个对象的任何属性和方法。它的作用仅仅是保证对象在 finalize 后，做某些事情。虚引用常见的使用场景是跟踪对象被垃圾回收的活动，当一个虚引用关联的对象被垃圾回收器回收之前会收到一条系统通知。

### Exception 和 Error的区别

Exception 和 Error 都继承于 Throwable，在 Java 中，只有 Throwable 类型的对象才能被 throw 或者 catch，它是异常处理机制的基本组成类型。

Exception 和 Error 体现了 Java 对不同异常情况的分类。Exception 是程序正常运行中，可以预料的意外情况，可能并且应该被捕获，进行相应的处理。

Error 是指在正常情况下，不大可能出现的情况，绝大部分 Error 都会使程序处于非正常、不可恢复的状态。既然是非正常，所以不便于也不需要捕获，常见的 OutOfMemoryError 就是 Error 的子类。

Exception 又分为 checked Exception 和 unchecked Exception。

- checked Exception 在代码里必须显式的进行捕获，这是编译器检查的一部分。
- unchecked Exception 也就是运行时异常，类似空指针异常、数组越界等，通常是可以避免的逻辑错误，具体根据需求来判断是否需要捕获，并不会在编译器强制要求。

### 线程的几个常见方法的比较

1. **Thread.sleep(long millis)**，一定是**当前线程**调用此方法，当前线程进入TIMED_WAITING状态，但不释放对象锁，millis后线程自动苏醒进入就绪状态。作用：给其它线程执行机会的最佳方式。
2. **Thread.yield()**，一定是**当前线程**调用此方法，当前线程放弃获取的CPU时间片，但不释放锁资源，由运行状态变为就绪状态，让OS再次选择线程。作用：让相同优先级的线程轮流执行，但并不保证一定会轮流执行。实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。Thread.yield()不会导致阻塞。该方法与sleep()类似，只是不能由用户指定暂停多长时间。
3. **thread.join()/thread.join(long millis)**，**当前线程里调用其它线程thread的join方法**，当前线程进入WAITING/TIMED_WAITING状态，当前线程不会释放已经持有的对象锁。线程thread执行完毕或者millis时间到，当前线程进入就绪状态。
4. **thread.interrupt()**,**当前线程里调用其它线程thread的interrupt()方法,中断指定的线程。**如果指定线程调用了wait()方法组或者join方法组在阻塞状态，那么指定线程会抛出InterruptedException
5. **Thread.interrupted**，一定是**当前线程**调用此方法，检查当前线程是否被设置了中断，**该方法会重置当前线程的中断标志**，返回当前线程是否被设置了中断。
6. **thread.isInterrupted()**，**当前线程里调用其它线程thread的isInterrupted()方法,返回指定线程是否被中断**
7. **object.wait()**，**当前线程**调用**对象的wait()**方法，当前线程释放对象锁，进入等待队列。依靠notify()/notifyAll()唤醒或者wait(long timeout) timeout时间到自动唤醒。
8. **object.notify()**唤醒在此**对象监视器上等待的单个线程，选择是任意性的**。notifyAll()唤醒在此对象监视器上等待的所有线程。

### Object.wait() / Object.notify() Object.notifyAll()

任意一个Java对象，都拥有一组监视器方法（定义在java.lang.Object上），主要包括wait()、

wait(long timeout)、notify()以及notifyAll()方法，这些方法与synchronized同步关键字配合，可以

实现等待/通知模式

1. 使用的前置条件当我们想要使用Object的监视器方法时，需要或者该Object的锁，代码如下所示**synchronized**(obj){ .... //1 obj.wait();//2 obj.wait(**long** millis);//2 ....//3 }一个线程获得obj的锁,做了一些时候事情之后，发现需要等待某些条件的发生，调用obj.wait()，该线程会释放obj的锁，并阻塞在上述的代码2处obj.wait()和obj.wait(long millis)的区别在于**obj.wait()是无限等待，直到obj.notify()或者obj.notifyAll()调用并唤醒该线程，该线程获取锁之后继续执行代码3obj.wait(long millis)是超时等待，我只等待long millis 后，该线程会自己醒来，醒来之后去获取锁，获取锁之后继续执行代码3obj.notify()是叫醒任意一个等待在该对象上的线程，该线程获取锁，线程状态从BLOCKED进入RUNNABLEobj.notifyAll()是叫醒所有等待在该对象上的线程，这些线程会去竞争锁，得到锁的线程状态从BLOCKED进入RUNNABLE，其他线程依然是BLOCKED,得到锁的线程执行代码3完毕后释放锁，其他线程继续竞争锁，如此反复直到所有线程执行完毕。synchronized**(obj){ .... //1 obj.notify();//2 obj.notifyAll();//2 }一个线程获得obj的锁,做了一些时候事情之后，某些条件已经满足，调用obj.notify()或者obj.notifyAll()，该线程会释放obj的锁，并叫醒在obj上等待的线程，obj.notify()和obj.notifyAll()的区别在于**obj.notify()叫醒在obj上等待的任意一个线程（由JVM决定)obj.notifyAll()叫醒在obj上等待的全部线程**
2. 使用范式synchronized(obj){ //判断条件，这里使用**while**，而不使用**ifwhile**(obj满足/不满足 某个条件){ obj.**wait**() } }放在while里面，是防止处于WAITING状态下线程监测的对象被别的原因调用了唤醒（notify或者notifyAll）方法，但是while里面的条件并没有满足（也可能当时满足了，但是由于别的线程操作后，又不满足了），就需要再次调用wait将其挂起

### 强软弱虚引用以及使用场景

**强引用：**

1. 正常创建的对象，只要引用存在，永远不会被GC回收，即使OOM

Object obj = new Object();

2. 如果要中断强引用和某个对象的关联，为其赋值null，这样GC就会在合适的时候回收对象

3. Vector类的clear()方法就是通过赋值null进行清除

**软引用**

1. 内存溢出之前进行回收，GC时内存不足时回收，如果内存足够就不回收

2. 使用场景：在内存足够的情况下进行缓存，提升速度，内存不足时JVM自动回收

3. 可以和引用队列ReferenceQueue联合使用，如果软引用所引用的对象被JVM回收，这个软引用就会被加入到与之关联的引用队列中

**弱引用**

1. 每次GC时回收，无论内存是否足够

2. 使用场景：a. ThreadLocalMap防止内存泄漏 b. 监控对象是否将要被回收

3. 弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被JVM回收，这个软引用就会被加入到与之关联的引用队列中

**虚引用**

1. 每次垃圾回收时都会被回收，主要用于监测对象是否已经从内存中删除

2. 虚引用必须和引用队列关联使用, 当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之 关联的引用队列中

3. 程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动

# Android

### 冷启动与热启动是什么，区别，如何优化，使用场景等

**app冷启动**： 当应用启动时，后台没有该应用的进程，这时系统会重新创建一个新的进程分配给该应用， 这个启动方式就叫做冷启动（后台不存在该应用进程）。冷启动因为系统会重新创建一个新的进程分配给它，所以会先创建和初始化Application类，再创建和初始化`MainActivity`类（包括一系列的测量、布局、绘制），最后显示在界面上。

**app热启动**： 当应用已经被打开， 但是被按下返回键、Home键等按键时回到桌面或者是其他程序的时候，再重新打开该app时， 这个方式叫做热启动（后台已经存在该应用进程）。热启动因为会从已有的进程中来启动，所以热启动就不会走Application这步了，而是直接走`MainActivity`（包括一系列的测量、布局、绘制），所以热启动的过程只需要创建和初始化一个`MainActivity`就行了，而不必创建和初始化`Application`

**冷启动的流程**

当点击app的启动图标时，安卓系统会从Zygote进程中fork创建出一个新的进程分配给该应用，之后会依次创建和初始化Application类、创建`MainActivity`类、加载主题样式Theme中的`windowBackground`等属性设置给`MainActivity`以及配置Activity层级上的一些属性、再inflate布局、当`onCreate`/`onStart`/`onResume`方法都走完了后最后才进行`contentView`的`measure`/`layout`/`draw`显示在界面上冷启动的生命周期简要流程：

Application构造方法 –> `attachBaseContext()`–>`onCreate` –>Activity构造方法 –> `onCreate()` –> 配置主体中的背景等操作 –>`onStart()` –> `onResume()` –> 测量、布局、绘制显示

冷启动的优化主要是视觉上的优化，解决白屏问题，提高用户体验，所以通过上面冷启动的过程。能做的优化如下：

> 1、减少onCreate()方法的工作量2、不要让Application参与业务的操作3、不要在Application进行耗时操作4、不要以静态变量的方式在Application保存数据5、减少布局的复杂度和层级6、减少主线程耗时

### 冷启动流程：

①点击桌面App图标，Launcher进程采用Binder IPC向system_server进程发起startActivity请求；

②system_server进程接收到请求后，向zygote进程发送创建进程的请求；

③Zygote进程fork出新的子进程，即App进程；

④App进程，通过Binder IPC向sytem_server进程发起attachApplication请求；

⑤system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向App进程发送scheduleLaunchActivity请求；

⑥App进程的binder线程（ApplicationThread）在收到请求后，通过handler向主线程发送LAUNCH_ACTIVITY消息；

⑦主线程在收到Message后，通过发射机制创建目标Activity，并回调Activity.onCreate()等方法。

⑧到此，App便正式启动，开始进入Activity生命周期，执行完onCreate/onStart/onResume方法，UI渲染结束后便可以看到App的主界面。

### Activity启动流程

1. Activity1调用startActivity，实际会调用Instrumentation类的execStartActivity方法，Instrumentation是系统用来监控Activity运行的一个类，Activity的整个生命周期都有它的影子。（1- 4）

2. 通过跨进程的binder调用，进入到ActivityManagerService中，其内部会处理Activity栈，通知Activity1 Pause，Activity1 执行Pause 后告知AMS。（5 - 29）

3. 在ActivityManagerService中的startProcessLocked中调用了Process.start()方法。并通过连接调用Zygote的native方法forkAndSpecialize，执行fork任务。之后再通过跨进程调用进入到Activity2所在的**进程**中。（30 - 36）

4. ApplicationThread是一个binder对象，其运行在binder线程池中，内部包含一个H类，该类继承于类Handler。主线程发起bind Application，AMS 会做一些配置工作，然后让主线程 bind ApplicationThread，ApplicationThread将启动Activity2的信息通过H对象发送给**主线程**。发送的消息是EXECUTE_TRANSACTION，消息体是一个 ClientTransaction，即 LaunchActivityItem。主线程拿到Activity2的信息后，调用Instrumentation类的newActivity方法，其内通过ClassLoader创建Activity2**实例**。（37 - 40）

5. 通知Activity2去performCreate。（41 - 最后）

注：现在发送的都是EXECUTE_TRANSACTION ，通过 TransactionExecutor 来执行 ClientTransaction, ClientTransaction 中包含各种 ClientTransactionItem，如 PauseActivityItem、LaunchActivityItem、StopActivityItem、ResumeActivityItem、DestroyActivityItem 等，这些Item的execute方法来处理相应的handle，如handlePauseActivity、handleLaunchActivity等，通知相应的Activity来perform。

### Activity生命周期

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c850daeb78?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c850daeb78?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

1.启动Activity：系统会先调用onCreate方法，然后调用onStart方法，最后调用onResume，Activity进入运行状态。

2.当前Activity被其他Activity覆盖其上或被锁屏：系统会调用onPause方法，暂停当前Activity的执行。

3.当前Activity由被覆盖状态回到前台或解锁屏：系统会调用onResume方法，再次进入运行状态。

4.当前Activity转到新的Activity界面或按Home键回到主屏，自身退居后台：系统会先调用onPause方法，然后调用onStop方法，进入停滞状态。

5.用户后退回到此Activity：系统会先调用onRestart方法，然后调用onStart方法，最后调用onResume方法，再次进入运行状态。

6.当前Activity处于被覆盖状态或者后台不可见状态，即第2步和第4步，系统内存不足，杀死当前Activity，而后用户退回当前Activity：再次调用onCreate方法、onStart方法、onResume方法，进入运行状态。

7.用户退出当前Activity：系统先调用onPause方法，然后调用onStop方法，最后调用onDestory方法，结束当前Activity。

### Activity四种启动模式

**standard**：标准模式：如果在mainfest中不设置就默认standard；standard就是新建一个Activity就在栈中新建一个activity实例；

**singleTop**：栈顶复用模式：与standard相比栈顶复用可以有效减少activity重复创建对资源的消耗，但是这要根据具体情况而定，不能一概而论；

**singleTask**：栈内单例模式，栈内只有一个activity实例，栈内已存activity实例，在其他activity中start这个activity，Android直接把这个实例上面其他activity实例踢出栈GC掉；

**singleInstance** :堆内单例：整个手机操作系统里面只有一个实例存在就是内存单例；

在singleTop、singleTask、singleInstance 中如果在应用内存在Activity实例，并且再次发生startActivity(Intent intent)回到Activity后,由于并不是重新创建Activity而是复用栈中的实例，因此Activity再获取焦点后并没调用onCreate、onStart，而是直接调用了onNewIntent(Intent intent)函数；

LauchMode	Instance

**standard**	邮件、mainfest中没有配置就默认标准模式

**singleTop**	登录页面、WXPayEntryActivity、WXEntryActivity 、推送通知栏

**singleTask**	程序模块逻辑入口:主页面（Fragment的containerActivity）、WebView页面、扫一扫页面、电商中：购物界面，确认订单界面，付款界面

**singleInstance**	系统Launcher、锁屏键、来电显示等系统应用

### activity横竖屏切换时activity的生命周期，view的生命周期

- 不配置configChanges时：切换横竖屏时生命周期各自都会走一遍
- 配置configChanges时：必须设置为android:configChanges="orientation|screenSize"时，才不会重走生命周期方法，只会回调onConfigurationChanged方法，注意，不配置configChanges或是配置了但不同时包含这两个值时，都会重走一遍生命周期方法，并且不会回调onConfigurationChanged方法。
- 另外重走生命周期方法时，还会调用**onSaveInstanceState() 与onRestoreIntanceState()**，资源相关的系统配置发生改变或者资源不足：例如屏幕旋转，当前Activity会销毁，并且**在onStop之前**回调**onSaveInstanceState**保存数据，在重新创建Activity的时候**在onStart之后**回调**onRestoreInstanceState**。其中Bundle数据会传到onCreate（不一定有数据）和onRestoreInstanceState（一定有数据）。用户或者程序员主动去销毁一个Activity的时候不会回调，其他情况都会调用，来保存界面信息。如代码中finish（）或用户按下back，不会回调。

### 五种进程

第一高：前台进程

前台进程是Android系统中最重要的进程，是与用户正在交互的进程。

第二高：可见进程

可见进程指部分程序界面能够被用户看见，却不在前台与用户交互。

第三高：服务进程

一个包含已启动服务的进程就是服务进程，服务没有用户界面，不与用户直接交互，但能够在后台长期运行，提供用户所关心的重要功能。

第四高：后台进程

如果一个进程不包含任何已经启动的服务，而且没有用户可见的Activity，则这个进程就是后台进程。

第五高：空进程

空进程是不包含任何活跃组件的进程。在系统资源紧张时会被首先清楚。

### startService和bindService的区别，生命周期以及使用场景

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c851a5cadd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c851a5cadd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**startService 和bindService 区别**

startService： onCreate -> onStartCommand -> onDestory ，在多次调用startService的时候，onCreate不重复执行，但是onStartCommand会执行。startService调用了这后，会一直存在，直到其调用了stopService。

bindService : onCreate -> onBind -> onUnbind -> onDestory，多次调用bindService，onCreate及onBind都只执行一次。它生命周期跟随其调用者，调用者释放的时候，必须对该Service解绑，当所有绑定全部取消后，系统即会销毁该服务。 bindService 的方式通过onServiceConnected方法，获取到Service对象，通过该对象可以直接操作到Service内部的方法，从而实现的Service 与调用者之间的交互。

**使用场景**

如果想要启动一个后台服务长期进行某项任务，那么使用startService

如果只是短暂的使用，那么使用bindService。

如果想启动一个后台服务长期进行任务，且这个过程中需要与调用者进行交互，那么可以两者同时使用，或者使用startService + BoardCast/ EventBus 等方法。

对于既使用startService，又使用bindService的情况，结束服务时需要注意的事项：

- Service的终止，需要unbindService和stopService都调用才行；

顺便提一下IntentService，与Service的区别在于它内部封装了一个工作线程，也就是说，在其内部onHandleIntent的代码都是在子线程里面工作的。

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c851c3faac?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c851c3faac?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### Android中IntentService有何优点

IntentService是一个通过Context.startService(Intent)启动可以处理异步请求的Service,使用时你只需要继承IntentService和重写其中的onHandleIntent(Intent)方法接收一个Intent对象,在适当的时候会停止自己(一般在工作完成的时候). 所有的请求的处理都在一个工作线程中完成,它们会交替执行(但不会阻塞主线程的执行),一次只能执行一个请求。

这是一个基于消息的服务,每次启动该服务并不是马上处理你的工作,而是首先会创建对应的Looper,Handler并且在MessageQueue中添加的附带客户Intent的Message对象,当Looper发现有Message的时候接着得到Intent对象通过在onHandleIntent((Intent)msg.obj)中调用你的处理程序.处理完后即会停止自己的服务.意思是Intent的生命周期跟你的处理的任务是一致的.所以这个类用下载任务中非常好,下载任务结束后服务自身就会结束退出.

### 进程间通信的方式有哪几种

AIDL 、广播、文件、socket、管道

### 广播静态注册和动态注册的区别

1. 动态注册广播不是常驻型广播，也就是说广播跟随 Activity 的生命周期。注意在 Activity 结束前，移除广播接收器。 静态注册是常驻型，也就是说当应用程序关闭后，如果有信息广播来，程序也会被系统调用自动运行。
2. 当广播为有序广播时：优先级高的先接收（不分静态和动态）。同优先级的广播接收器，动态优先于静态
3. 同优先级的同类广播接收器，静态：先扫描的优先于后扫描的，动态：先注册的优先于后注册的。
4. 当广播为默认广播时：无视优先级，动态广播接收器优先于静态广播接收器。同优先级的同类广播接收器，静态：先扫描的优先于后扫描的，动态：先注册的优先于后册的。

### Android 性能优化工具使用（这个问题建议配合Android中的性能优化）

Android 中常用的性能优化工具包括这些：Android Studio 自带的 Android Profiler、LeakCanary、BlockCanary

Android 自带的 Android Profiler 其实就很好用，Android Profiler 可以检测三个方面的性能问题：CPU、MEMORY、NETWORK。

LeakCanary 是一个第三方的检测内存泄漏的库，我们的项目集成之后 LeakCanary 会自动检测应用运行期间的内存泄漏，并将之输出给我们。

BlockCanary 也是一个第三方检测UI卡顿的库，项目集成后Block也会自动检测应用运行期间的UI卡顿，并将之输出给我们。

### Android中的类加载器

- PathClassLoader，只能加载系统中已经安装过的 apk
- DexClassLoader，可以加载 `jar/apk/dex`，可以从 SD卡中加载未安装的 apk

### Android中的动画有哪几类，它们的特点和区别是什么

Android中动画大致分为3类：帧动画、补间动画（Tween Animation）、属性动画（Property Animation）。

- 帧动画：通过xml配置一组图片，动态播放。很少会使用。
- 补间动画（Tween Animation）：大致分为旋转、透明、缩放、位移四类操作。很少会使用。
- 属性动画（Property Animation）：属性动画是现在使用的最多的一种动画，它比补间动画更加强大。属性动画大致分为两种使用类型，分别是 ViewPropertyAnimator 和 ObjectAnimator。前者适合一些通用的动画，比如旋转、位移、缩放和透明，使用方式也很简单通过 `View.animate()` 即可得到 ViewPropertyAnimator，之后进行相应的动画操作即可。后者适合用于为我们的自定义控件添加动画，当然首先我们应该在自定义 View 中添加相应的 `getXXX()` 和 `setXXX()` 相应属性的 getter 和 setter 方法，这里需要注意的是在 setter 方法内改变了自定义 View 中的属性后要调用 `invalidate()` 来刷新View的绘制。之后调用 `ObjectAnimator.of` 属性类型()返回一个 ObjectAnimator，调用 `start()` 方法启动动画即可。

补间动画与属性动画的区别：

- 补间动画是父容器不断的绘制 view，看起来像移动了效果,其实 view 没有变化，还在原地。
- 是通过不断改变 view 内部的属性值，真正的改变 view。

**TimeInterpolator（时间插值器）**

**作用：**根据时间流逝的百分比计算出当前属性值改变的百分比

系统已有的插值器：

- **LinearInterpolator（线性插值器）：**匀速动画。
- **AccelerateDecelerateInterpolator（加速减速插值器）：**动画两头慢，中间快。
- **DecelerateInterpolator（减速插值器）：**动画越来越慢。

**TypeEvaluator（类型估值算法，即估值器）：**

**作用：**根据当前属性改变的百分比来计算改变后的属性值。

系统已有的估值器：

- **IntEvaluator：**针对整型属性
- **FloatEvaluator：**针对浮点型属性
- **ArgbEvaluator：**针对Color属性

### Handler 机制

说到 Handler，就不得不提与之密切相关的这几个类：Message、MessageQueue，Looper。

- **Message。**Message 中有两个成员变量值得关注：target 和 callback。
- target 其实就是发送消息的 Handler 对象
- callback 是当调用 `handler.post(runnable)` 时传入的 Runnable 类型的任务。post 事件的本质也是创建了一个 Message，将我们传入的这个 runnable 赋值给创建的Message的 callback 这个成员变量。
- **MessageQueue。** 消息队列很明显是存放消息的队列，值得关注的是 MessageQueue 中的 `next()` 方法，它会返回下一个待处理的消息。
- **Looper。**Looper 消息轮询器其实是连接 Handler 和消息队列的核心。首先我们都知道，如果想要在一个线程中创建一个 Handler，首先要通过`Looper.prepare()`创建 Looper，之后还得调用`Looper.loop()`开启轮询。我们着重看一下这两个方法。
- **`prepare()`。** 这个方法做了两件事：首先通过`ThreadLocal.get()`获取当前线程中的Looper,如果不为空，则会抛出一个RunTimeException，意思是一个线程不能创建2个Looper。如果为null则执行下一步。第二步是创建了一个Looper，并通过 `ThreadLocal.set(looper)。`将我们创建的Looper与当前线程绑定。这里需要提一下的是消息队列的创建其实就发生在Looper的构造方法中。
- **`loop()`。** 这个方法开启了整个事件机制的轮询。它的本质是开启了一个死循环，不断的通过 `MessageQueue的next()`方法获取消息。拿到消息后会调用 `msg.target.dispatchMessage()`来做处理。其实我们在说到 Message 的时候提到过，`msg.target` 其实就是发送这个消息的 handler。这句代码的本质就是调用 `handler的dispatchMessage()。`
- **Handler。**上面做了这么多铺垫，终于到了最重要的部分。Handler 的分析着重在两个部分：发送消息和处理消息。*发送消息。其实发送消息除了 sendMessage 之外还有 sendMessageDelayed 和 post 以及 postDelayed 等等不同的方式。但它们的本质都是调用了 sendMessageAtTime。在 sendMessageAtTime 这个方法中调用了 enqueueMessage。在 enqueueMessage 这个方法中做了两件事：通过`msg.target = this`实现了消息与当前 handler 的绑定。然后通过`queue.enqueueMessage`实现了消息入队。
- **处理消息。** 消息处理的核心其实就是`dispatchMessage()`这个方法。这个方法里面的逻辑很简单，先判断 `msg.callback` 是否为 null，如果不为空则执行这个 runnable。如果为空则会执行我们的`handleMessage`方法。

### Handler面试知识点

[www.yuque.com/docs/share/…](https://www.yuque.com/docs/share/0adad94b-b20c-437d-a1fc-f8d329517f37?#)

### 子线程更新ui会怎么样，为什么不让子线程更新ui,在oncreate里用子 线程更新ui为什么不会报错

**`ActivityThread.handleResumeActivity（） 中初始化了ViewRootImpl 然后执行 requestLayout()进行线程校验`**

```java
if (r.window == null && !a.mFinished && willBeVisible) {
            r.window = r.activity.getWindow();
            View decor = r.window.getDecorView();
            decor.setVisibility(View.INVISIBLE);
            ViewManager wm = a.getWindowManager();
            WindowManager.LayoutParams l = r.window.getAttributes();
            a.mDecor = decor;
            l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
            l.softInputMode |= forwardBit;
            if (r.mPreserveWindow) {
                a.mWindowAdded = true;
                r.mPreserveWindow = false;
                // Normally the ViewRoot sets up callbacks with the Activity
                // in addView->ViewRootImpl#setView. If we are instead reusing
                // the decor view we have to notify the view root that the
                // callbacks may have changed.
                ViewRootImpl impl = decor.getViewRootImpl();
                if (impl != null) {
                    impl.notifyChildRebuilt();
                }
            }
            if (a.mVisibleFromClient) {
                if (!a.mWindowAdded) {
                    a.mWindowAdded = true;
                    wm.addView(decor, l);
                } else {
								
								}
						}
```

### Android 性能优化

Android 中的性能优化在我看来分为以下几个方面：内存优化、布局优化、网络优化、安装包优化。

- **内存优化：** 下一个问题就是。
- **布局优化：**布局优化的本质就是减少 View 的层级。常见的布局优化方案如下
- 在 LinearLayout 和 RelativeLayout 都可以完成布局的情况下优先选择 RelativeLayout，可以减少 View 的层级
- 将常用的布局组件抽取出来使用 `\< include \>`标签
- 通过 `\< ViewStub \>`标签来加载不常用的布局
- 使用 `\< Merge \>`标签来减少布局的嵌套层次
- **网络优化：**常见的网络优化方案如下
- 尽量减少网络请求，能够合并的就尽量合并
- 避免 DNS 解析，根据域名查询可能会耗费上百毫秒的时间，也可能存在DNS劫持的风险。可以根据业务需求采用增加动态更新 IP 的方式，或者在 IP 方式访问失败时切换到域名访问方式。
- 大量数据的加载采用分页的方式
- 网络数据传输采用 GZIP 压缩
- 加入网络数据的缓存，避免频繁请求网络
- 上传图片时，在必要的时候压缩图片
- **安装包优化：**安装包优化的核心就是减少 apk 的体积，常见的方案如下
- 使用混淆，可以在一定程度上减少 apk 体积，但实际效果微乎其微
- 减少应用中不必要的资源文件，比如图片，在不影响 APP 效果的情况下尽量压缩图片，有一定的效果
- 在使用了 SO 库的时候优先保留 v7 版本的 SO 库，删掉其他版本的SO库。原因是在 2018 年，v7 版本的 SO 库可以满足市面上绝大多数的要求，可能八九年前的手机满足不了，但我们也没必要去适配老掉牙的手机。实际开发中减少 apk 体积的效果是十分显著的，如果你使用了很多 SO 库，比方说一个版本的SO库一共 10M，那么只保留 v7 版本，删掉 armeabi 和 v8 版本的 SO 库，一共可以减少 20M 的体积。

### Android 内存优化

Android的内存优化在我看来分为两点：**避免内存泄漏、扩大内存，其实就是开源节流。**

其实内存泄漏的本质就是较长生命周期的对象引用了较短生命周期的对象。

### 常见的内存泄漏

- **单例模式导致的内存泄漏。** 最常见的例子就是创建这个单例对象需要传入一个 Context，这时候传入了一个 Activity 类型的 Context，由于单例对象的静态属性，导致它的生命周期是从单例类加载到应用程序结束为止，所以即使已经 finish 掉了传入的 Activity，由于我们的单例对象依然持有 Activity 的引用，所以导致了内存泄漏。解决办法也很简单，不要使用 Activity 类型的 Context，使用 Application 类型的 Context 可以避免内存泄漏。
- **静态变量导致的内存泄漏。** 静态变量是放在方法区中的，它的生命周期是从类加载到程序结束，可以看到静态变量生命周期是非常久的。最常见的因静态变量导致内存泄漏的例子是我们在 Activity 中创建了一个静态变量，而这个静态变量的创建需要传入 Activity 的引用 this。在这种情况下即使 Activity 调用了 finish 也会导致内存泄漏。原因就是因为这个静态变量的生命周期几乎和整个应用程序的生命周期一致，它一直持有 Activity 的引用，从而导致了内存泄漏。
- **非静态内部类导致的内存泄漏。**非静态内部类导致内存泄漏的原因是非静态内部类持有外部类的引用，最常见的例子就是在 Activity 中使用 Handler 和 Thread 了。使用非静态内部类创建的 Handler 和 Thread 在执行延时操作的时候会一直持有当前Activity的引用，如果在执行延时操作的时候就结束 Activity，这样就会导致内存泄漏。解决办法有两种：第一种是使用静态内部类，在静态内部类中使用弱引用调用Activity。第二种方法是在 Activity 的 onDestroy 中调用 `handler.removeCallbacksAndMessages` 来取消延时事件。
- 使用资源未及时关闭导致的内存泄漏。常见的例子有：操作各种数据流未及时关闭，操作 Bitmap 未及时 recycle 等等。
- 使用第三方库未能及时解绑。有的三方库提供了注册和解绑的功能，最常见的就 EventBus 了，我们都知道使用 EventBus 要在 onCreate 中注册，在 onDestroy 中解绑。如果没有解绑的话，EventBus 其实是一个单例模式，他会一直持有 Activity 的引用，导致内存泄漏。同样常见的还有 RxJava，在使用 Timer 操作符做了一些延时操作后也要注意在 onDestroy 方法中调用 `disposable.dispose()`来取消操作。
- 属性动画导致的内存泄漏。常见的例子就是在属性动画执行的过程中退出了 Activity，这时 View 对象依然持有 Activity 的引用从而导致了内存泄漏。解决办法就是在 onDestroy 中调用动画的 cancel 方法取消属性动画。
- WebView 导致的内存泄漏。WebView 比较特殊，即使是调用了它的 destroy 方法，依然会导致内存泄漏。其实避免WebView导致内存泄漏的最好方法就是让WebView所在的Activity处于另一个进程中，当这个 Activity 结束时杀死当前 WebView 所处的进程即可，我记得阿里钉钉的 WebView 就是另外开启的一个进程，应该也是采用这种方法避免内存泄漏。

**扩大内存**

为什么要扩大我们的内存呢？有时候我们实际开发中不可避免的要使用很多第三方商业的 SDK，这些 SDK 其实有好有坏，大厂的 SDK 可能内存泄漏会少一些，但一些小厂的 SDK 质量也就不太靠谱一些。那应对这种我们无法改变的情况，最好的办法就是扩大内存。

扩大内存通常有两种方法：一个是在清单文件中的 Application 下添加`largeHeap="true"`这个属性，另一个就是同一个应用开启多个进程来扩大一个应用的总内存空间。第二种方法其实就很常见了，比方说我使用过个推的 S DK，个推的 Service 其实就是处在另外一个单独的进程中。

Android 中的内存优化总的来说就是开源和节流，开源就是扩大内存，节流就是避免内存泄漏。

### Binder 机制

在Linux中，为了避免一个进程对其他进程的干扰，进程之间是相互独立的。在一个进程中其实还分为用户空间和内核空间。这里的隔离分为两个部分，进程间的隔离和进程内的隔离。

既然进程间存在隔离，那其实也是存在着交互。进程间通信就是 IPC，用户空间和内核空间的通信就是系统调用。

Linux 为了保证独立性和安全性，进程之间不能直接相互访问，Android 是基于 Linux 的，所以也是需要解决进程间通信的问题。

其实 Linux 进程间通信有很多方式，比如管道、socket 等等。为什么 Android 进程间通信采用了Binder而不是 Linux

已有的方式，主要是有这么两点考虑：**性能和安全**

- **性能。** 在移动设备上对性能要求是比较严苛的。Linux传统的进程间通信比如管道、socket等等进程间通信是需要复制两次数据，而Binder则只需要一次。所以Binder在性能上是优于传统进程通信的。
- **安全。** 传统的 Linux 进程通信是不包含通信双方的身份验证的，这样会导致一些安全性问题。而Binder机制自带身份验证，从而有效的提高了安全性。

Binder 是基于 CS 架构的，有四个主要组成部分。

- **Client。** 客户端进程。
- **Server。** 服务端进程。
- **ServiceManager。** 提供注册、查询和返回代理服务对象的功能。
- **Binder 驱动。** 主要负责建立进程间的 Binder 连接，进程间的数据交互等等底层操作。

Binder 机制主要的流程是这样的：

- 服务端通过Binder驱动在 ServiceManager 中注册我们的服务。
- 客户端通过Binder驱动查询在 ServiceManager 中注册的服务。
- ServiceManager 通过 inder 驱动返回服务端的代理对象。
- 客户端拿到服务端的代理对象后即可进行进程间通信。

### LruCache的原理

LruCache 的核心原理就是对 LinkedHashMap 的有效利用，它的内部存在一个 LinkedHashMap 成员变量。值得我们关注的有四个方法：**构造方法、get、put、trimToSize。**

- **构造方法：** 在 LruCache 的构造方法中做了两件事，设置了 maxSize、创建了一个 LinkedHashMap。这里值得注意的是 LruCache 将 LinkedHashMap的accessOrder 设置为了 true，accessOrder 就是遍历这个LinkedHashMap 的输出顺序。true 代表按照访问顺序输出，false代表按添加顺序输出，因为通常都是按照添加顺序输出，所以 accessOrder 这个属性默认是 false，但我们的 LruCache 需要按访问顺序输出，所以显式的将 accessOrder 设置为 true。
- **get方法：** 本质上是调用 LinkedHashMap 的 get 方法，由于我们将 accessOrder 设置为了 true，所以每调用一次get方法，就会将我们访问的当前元素放置到这个LinkedHashMap的尾部。
- **put方法：** 本质上也是调用了 LinkedHashMap 的 put 方法，由于 LinkedHashMap 的特性，每调用一次 put 方法，也会将新加入的元素放置到 LinkedHashMap 的尾部。添加之后会调用 trimToSize 方法来保证添加后的内存不超过 maxSize。
- **trimToSize方法：** trimToSize 方法的内部其实是开启了一个 while(true)的死循环，不断的从 LinkedHashMap 的首部删除元素，直到删除之后的内存小于 maxSize 之后使用 break 跳出循环。

其实到这里我们可以总结一下，为什么这个算法叫 **最近最少使用** 算法呢？原理很简单，我们的每次 put 或者get都可以看做一次访问，由于 LinkedHashMap 的特性，会将每次访问到的元素放置到尾部。当我们的内存达到阈值后，会触发 trimToSize 方法来删除 LinkedHashMap 首部的元素，直到当前内存小于 maxSize。为什么删除首部的元素，原因很明显：我们最近经常访问的元素都会放置到尾部，那首部的元素肯定就是 **最近最少使用** 的元素了，因此当内存不足时应当优先删除这些元素。

### DiskLruCache原理

### 设计一个图片的异步加载框架

设计一个图片加载框架，肯定要用到图片加载的三级缓存的思想。三级缓存分为内存缓存、本地缓存和网络缓存。

内存缓存：将Bitmap缓存到内存中，运行速度快，但是内存容量小。 本地缓存：将图片缓存到文件中，速度较慢，但容量较大。 网络缓存：从网络获取图片，速度受网络影响。

如果我们设计一个图片加载框架，流程一定是这样的：

- 拿到图片url后首先从内存中查找BItmap，如果找到直接加载。
- 内存中没有找到，会从本地缓存中查找，如果本地缓存可以找到，则直接加载。
- 内存和本地都没有找到，这时会从网络下载图片，下载到后会加载图片，并且将下载到的图片放到内存缓存和本地缓存中。

上面是一些基本的概念，如果是具体的代码实现的话，大概需要这么几个方面的文件：

- 首先需要确定我们的内存缓存，这里一般用的都是 LruCache。
- 确定本地缓存，通常用的是 DiskLruCache，这里需要注意的是图片缓存的文件名一般是 url 被 MD5 加密后的字符串，为了避免文件名直接暴露图片的 url。
- 内存缓存和本地缓存确定之后，需要我们创建一个新的类 MemeryAndDiskCache，当然，名字随便起，这个类包含了之前提到的 LruCache 和 DiskLruCache。在 MemeryAndDiskCache 这个类中我们定义两个方法，一个是 getBitmap，另一个是 putBitmap，对应着图片的获取和缓存，内部的逻辑也很简单。getBitmap中按内存、本地的优先级去取 BItmap，putBitmap 中先缓存内存，之后缓存到本地。
- 在缓存策略类确定好之后，我们创建一个 ImageLoader 类，这个类必须包含两个方法，一个是展示图片 `displayImage(url,imageView)`，另一个是从网络获取图片`downloadImage(url,imageView)`。在展示图片方法中首先要通过 `ImageView.setTag(url)`，将 url 和 imageView 进行绑定，这是为了避免在列表中加载网络图片时会由于ImageView的复用导致的图片错位的 bug。之后会从 MemeryAndDiskCache 中获取缓存，如果存在，直接加载；如果不存在，则调用从网络获取图片这个方法。从网络获取图片方法很多，这里我一般都会使用 `OkHttp+Retrofit`。当从网络中获取到图片之后，首先判断一下`imageView.getTag()`与图片的 url 是否一致，如果一致则加载图片，如果不一致则不加载图片，通过这样的方式避免了列表中异步加载图片的错位。同时在获取到图片之后会通过 MemeryAndDiskCache 来缓存图片。

### Android中的事件分发机制

在我们的手指触摸到屏幕的时候，事件其实是通过 `Activity -> ViewGroup -> View` 这样的流程到达最后响应我们触摸事件的 View。

说到事件分发，必不可少的是这几个方法：`dispatchTouchEvent()、onInterceptTouchEvent()、onTouchEvent。`接下来就按照`Activity -> ViewGroup -> View` 的流程来大致说一下事件分发机制。

我们的手指触摸到屏幕的时候，会触发一个 Action_Down 类型的事件，当前页面的 Activity 会首先做出响应，也就是说会走到 Activity 的 `dispatchTouchEvent()` 方法内。在这个方法内部简单来说是这么一个逻辑：

- 调用 `getWindow.superDispatchTouchEvent()。`
- 如果上一步返回 true，直接返回 true；否则就 return 自己的 `onTouchEvent()。` 这个逻辑很好理解，`getWindow().superDispatchTouchEvent()` 如果返回 true 代表当前事件已经被处理，无需调用自己的 onTouchEvent；否则代表事件并没有被处理，需要 Activity 自己处理，也就是调用自己的 onTouchEvent。

`getWindow()`方法返回了一个 Window 类型的对象，这个我们都知道，在 Android 中，PhoneWindow 是Window 的唯一实现类。所以这句本质上是调用了``PhoneWindow中的superDispatchTouchEvent()。`

而在 PhoneWindow 的这个方法中实际调用了`mDecor.superDispatchTouchEvent(event)`。这个 mDecor 就是 DecorView，它是 FrameLayout 的一个子类，在 DecorView 中的 `superDispatchTouchEvent()` 中调用的是 `super.dispatchTouchEvent()`。到这里就很明显了，DecorView 是一个 FrameLayout 的子类，FrameLayout 是一个 ViewGroup 的子类，本质上调用的还是 `ViewGroup的dispatchTouchEvent()`。

分析到这里，我们的事件已经从 Activity 传递到了 ViewGroup，接下来我们来分析下 ViewGroup 中的这几个事件处理方法。

在 ViewGroup 中的 `dispatchTouchEvent()`中的逻辑大致如下：

- 通过 `onInterceptTouchEvent()` 判断当前 ViewGroup 是否拦截事件，默认的 ViewGroup 都是不拦截的；
- 如果拦截，则 return 自己的 `onTouchEvent()`；
- 如果不拦截，则根据 `child.dispatchTouchEvent()`的返回值判断。如果返回 true，则 return true；否则 return 自己的 `onTouchEvent()`，在这里实现了未处理事件的向上传递。

通常情况下 ViewGroup 的 `onInterceptTouchEvent()`都返回 false，也就是不拦截。这里需要注意的是事件序列，比如 Down 事件、Move 事件......Up事件，从 Down 到 Up 是一个完整的事件序列，对应着手指从按下到抬起这一系列的事件，如果 ViewGroup 拦截了 Down 事件，那么后续事件都会交给这个 ViewGroup的onTouchEvent。如果 ViewGroup 拦截的不是 Down 事件，那么会给之前处理这个 Down 事件的 View 发送一个 Action_Cancel 类型的事件，通知子 View 这个后续的事件序列已经被 ViewGroup 接管了，子 View 恢复之前的状态即可。

这里举一个常见的例子：在一个 Recyclerview 钟有很多的 Button，我们首先按下了一个 button，然后滑动一段距离再松开，这时候 Recyclerview 会跟着滑动，并不会触发这个 button 的点击事件。这个例子中，当我们按下 button 时，这个 button 接收到了 Action_Down 事件，正常情况下后续的事件序列应该由这个 button处理。但我们滑动了一段距离，这时 Recyclerview 察觉到这是一个滑动操作，拦截了这个事件序列，走了自身的 `onTouchEvent()`方法，反映在屏幕上就是列表的滑动。而这时 button 仍然处于按下的状态，所以在拦截的时候需要发送一个 Action_Cancel 来通知 button 恢复之前状态。

事件分发最终会走到 View 的 `dispatchTouchEvent()`中。在 View 的 `dispatchTouchEvent()` 中没有 `onInterceptTouchEvent()`，这也很容易理解，View 不是 ViewGroup，不会包含其他子 View，所以也不存在拦截不拦截这一说。忽略一些细节，View 的 `dispatchTouchEvent()`中直接 return 了自己的 `onTouchEvent()`。如果 `onTouchEvent()`返回 true 代表事件被处理，否则未处理的事件会向上传递，直到有 View 处理了事件或者一直没有处理，最终到达了 Activity 的 `onTouchEvent()` 终止。

这里经常有人问 onTouch 和 onTouchEvent 的区别。首先，这两个方法都在 View 的 `dispatchTouchEvent()`中，是这么一个逻辑：

- 如果 touchListener 不为 null，并且这个 View 是 enable 的，而且 onTouch 返回的是 true，满足这三个条件时会直接 return true，不会走 `onTouchEvent()`方法。
- 上面只要有一个条件不满足，就会走到 `onTouchEvent()`方法中。所以 onTouch 的顺序是在 onTouchEvent 之前的。

## View

### 绘制流程

视图绘制的起点在 ViewRootImpl 类的 `performTraversals()`方法，在这个方法内其实是按照顺序依次调用了 `mView.measure()、mView.layout()、mView.draw()`

View的绘制流程分为3步：测量、布局、绘制，分别对应3个方法 measure、layout、draw。

- **测量阶段。**measure 方法会被父 View 调用，在measure 方法中做一些优化和准备工作后会调用 onMeasure 方法进行实际的自我测量。onMeasure方法在View和ViewGroup做的事情是不一样的：
- **View。** View 中的 onMeasure 方法会计算自己的尺寸并通过 setMeasureDimension 保存。
- **ViewGroup。** ViewGroup 中的 onMeasure 方法会调用所有子 iew的measure 方法进行自我测量并保存。然后通过子View的尺寸和位置计算出自己的尺寸并保存。
- **布局阶段。**layout 方法会被父View调用，layout 方法会保存父 View 传进来的尺寸和位置，并调用 onLayout 进行实际的内部布局。onLayout 在 View 和 ViewGroup 中做的事情也是不一样的：
- **View。** 因为 View 是没有子 View 的，所以View的onLayout里面什么都不做。
- **ViewGroup。** ViewGroup 中的 onLayout 方法会调用所有子 View 的 layout 方法，把尺寸和位置传给他们，让他们完成自我的内部布局。
- **绘制阶段。**draw 方法会做一些调度工作，然后会调用 onDraw 方法进行 View 的自我绘制。draw 方法的调度流程大致是这样的：
- **绘制背景。**对应 `drawBackground(Canvas)`方法。
- **绘制主体。**对应 `onDraw(Canvas)`方法。
- **绘制子View。** 对应 `dispatchDraw(Canvas)`方法。
- **绘制滑动相关和前景。** 对应 `onDrawForeground(Canvas)`。

### MeasureSpec

MeasureSpec 是 View 的测量规则。通常父控件要测量子控件的时候，会传给子控件 widthMeasureSpec 和 heightMeasureSpec 这两个 int 类型的值。这个值里面包含两个信息，**SpecMode** 和 **SpecSize**。一个 int 值怎么会包含两个信息呢？我们知道 int 是一个4字节32位的数据，在这两个 int 类型的数据中，前面高2位是 **SpecMode** ，后面低30位代表了 **SpecSize**。

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c85261aca8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c85261aca8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

mode 有三种类型：

```
UNSPECIFIED、EXACTLY、AT_MOST
```

[Untitled](https://www.notion.so/055e40355f904647acc90b0556ede4f5)

我们怎么从一个 int 值里面取出两个信息呢？别担心，在 View 内部有一个 MeasureSpec 类。这个类已经给我们封装好了各种方法:

```java
//将 Size 和 mode 组合成一个 int 值
int measureSpec = MeasureSpec.makeMeasureSpec(size,mode);
//获取 size 大小
int size = MeasureSpec.getSize(measureSpec);
//获取 mode 类型
int mode = MeasureSpec.getMode(measureSpec);
```

具体实现细节，可以查看源码

### DecorView 的 measureSpec 计算逻辑

可能我们会有疑问，如果所有子控件的 measureSpec 都是父控件结合自身的 measureSpec 和子 View 的 LayoutParams 来生成的。那么作为视图的顶级父类 DecorView 怎么获取自己的 measureSpec 呢？下面我们来分析源码：（以下源码有所删减）

```java
//ViewRootImpl 类
private void performTraversals() {
    //获取 DecorView 宽度的 measureSpec 
    int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
    //获取 DecorView 高度的 measureSpec
    int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);
    // Ask host how big it wants to be
    //开始执行测量
    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
}
复制代码
```

```java
//ViewRootImpl 类
private static int getRootMeasureSpec(int windowSize, int rootDimension) {
    int measureSpec;
    switch (rootDimension) {
        case ViewGroup.LayoutParams.MATCH_PARENT:
            // Window can't resize. Force root view to be windowSize.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
            break;
        case ViewGroup.LayoutParams.WRAP_CONTENT:
            // Window can resize. Set max size for root view.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
            break;
        default:
            // Window wants to be an exact size. Force root view to be that size.
            measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
            break;
    }
    return measureSpec;
}
```

windowSize 是 widow 的宽高大小，所以我们可以看出 DecorView 的 measureSpec 是根据 window 的宽高大小和自身的 LayoutParams 来生成的。

### 事件冲突解决思路与方案

两种方案：外部和内部

### 为什么说android UI操作不是线程安全的

可能在非UI线程中刷新界面的时候，UI线程(或者其他非UI线程)也在刷新界面，这样就导致多个界面刷新的操作不能同步，导致线程不安全。

### ANR发生的原因总结和解决办法

ANR的全称是application not responding，是指应用程序未响应，Android系统对于一些事件需要在一定的时间范围内完成，如果超过预定时间能未能得到有效响应或者响应时间过长，都会造成ANR。一般地，这时往往会弹出一个提示框，告知用户当前xxx未响应，用户可选择继续等待或者Force Close。

首先ANR的发生是有条件限制的，分为以下三点：

只有主线程才会产生ANR，主线程就是UI线程；

必须发生某些输入事件或特定操作，比如按键或触屏等输入事件，在BroadcastReceiver或Service的各个生命周期调用函数；

上述事件响应超时，不同的context规定的上限时间不同

a.主线程对输入事件5秒内没有处理完毕

b.主线程在执行BroadcastReceiver的onReceive()函数时10秒内没有处理完毕

c.主线程在前台Service的各个生命周期函数时20秒内没有处理完毕（后台Service200s）

**那么导致ANR的根本原因是什么呢？简单的总结有以下两点：**

1.主线程执行了耗时操作，比如数据库操作或网络编程,I/O操作

2.其他进程（就是其他程序）占用CPU导致本进程得不到CPU时间片，比如其他进程的频繁读写操作可能会导致这个问题。

细分的话，导致ANR的原因有如下几点：

耗时的网络访问

大量的数据读写

数据库操作

硬件操作（比如camera)

调用thread的join()方法、sleep()方法、wait()方法或者等待线程锁的时候

service binder的数量达到上限

system server中发生WatchDog ANR

service忙导致超时无响应

其他线程持有锁，导致主线程等待超时

其它线程终止或崩溃导致主线程一直等待

**那么如何避免ANR的发生呢或者说ANR的解决办法是什么呢？**

1.避免在主线程执行耗时操作，所有耗时操作应新开一个子线程完成，然后再在主线程更新UI。

2.BroadcastReceiver要执行耗时操作时应启动一个service，将耗时操作交给service来完成。

3.避免在Intent Receiver里启动一个Activity，因为它会创建一个新的画面，并从当前用户正在运行的程序上抢夺焦点。如果你的应用程序在响应Intent广 播时需要向用户展示什么，你应该使用Notification Manager来实现。

### Android 源码中常见的设计模式以及自己在开发中常用的设计模式

[blog.csdn.net/zxc123e/art…](https://blog.csdn.net/zxc123e/article/details/55213252)

### Android与 js 是如何交互的

在 Android 中，Android 与js 的交互分为两个方面：Android 调用 js 里的方法、js 调用 Android 中的方法。

- **Android调js。**Android 调 js 有两种方法：
- **WebView.loadUrl("javascript:js中的方法名")。** 这种方法的优点是很简洁，缺点是没有返回值，如果需要拿到js方法的返回值则需要js调用Android中的方法来拿到这个返回值。
- **WebView.evaluateJavaScript("javascript:js中的方法名",ValueCallback)。** 这种方法比 loadUrl 好的是可以通过 ValueCallback 这个回调拿到 js方法的返回值。缺点是这个方法 Android4.4 才有，兼容性较差。不过放在 2018 年来说，市面上绝大多数 App 都要求最低版本是 4.4 了，所以我认为这个兼容性问题不大。
- **js 调 Android。**js 调 Android有三种方法：
- **`WebView.addJavascriptInterface()。`** 这是官方解决 js 调用 Android 方法的方案，需要注意的是要在供 js 调用的 Android 方法上加上 **@JavascriptInterface** 注解，以避免安全漏洞。这种方案的缺点是 Android4.2 以前会有安全漏洞，不过在 4.2 以后已经修复了。同样，在 2018 年来说，兼容性问题不大。
- **重写 `WebViewClient的shouldOverrideUrlLoading()`方法来拦截url，** 拿到 url 后进行解析，如果符合双方的规定，即可调用 Android 方法。优点是避免了 Android4.2 以前的安全漏洞，缺点也很明显，无法直接拿到调用 Android 方法的返回值，只能通过 Android 调用 js 方法来获取返回值。
- 重写 WebChromClient 的 `onJsPrompt()` 方法，同前一个方式一样，拿到 url 之后先进行解析，如果符合双方规定，即可调用Android方法。最后如果需要返回值，通过 `result.confirm("Android方法返回值")` 即可将 Android 的返回值返回给 js。方法的优点是没有漏洞，也没有兼容性限制，同时还可以方便的获取 Android 方法的返回值。其实这里需要注意的是在 WebChromeClient 中除 了 onJsPrompt 之外还有 onJsAlert 和 onJsConfirm 方法。那么为什么不选择另两个方法呢？原因在于 onJsAlert 是没有返回值的，而 onJsConfirm 只有 true 和 false 两个返回值，同时在前端开发中 prompt 方法基本不会被调用，所以才会采用 onJsPrompt。

### SparseArray 原理

SparseArray，通常来讲是 Android 中用来替代 HashMap 的一个数据结构。 准确来讲，是用来替换key为 Integer 类型，value为Object 类型的HashMap。需要注意的是 SparseArray 仅仅实现了 Cloneable 接口，所以不能用Map来声明。 从内部结构来讲，SparseArray 内部由两个数组组成，一个是 `int[]`类型的 mKeys，用来存放所有的键；另一个是 `Object[]`类型的 mValues，用来存放所有的值。 最常见的是拿 SparseArray 跟HashMap 来做对比，由于 SparseArray 内部组成是两个数组，所以占用内存比 HashMap 要小。我们都知道，增删改查等操作都首先需要找到相应的键值对，而 SparseArray 内部是通过二分查找来寻址的，效率很明显要低于 HashMap 的常数级别的时间复杂度。提到二分查找，这里还需要提一下的是二分查找的前提是数组已经是排好序的，没错，SparseArray 中就是按照key进行升序排列的。 综合起来来说，SparseArray 所占空间优于 HashMap，而效率低于 HashMap，是典型的时间换空间，适合较小容量的存储。 从源码角度来说，我认为需要注意的是 `SparseArray的remove()、put()`和 `gc()`方法。

- **`remove()`。** SparseArray 的 `remove()` 方法并不是直接删除之后再压缩数组，而是将要删除的 value 设置为 DELETE 这个 SparseArray 的静态属性，这个 DELETE 其实就是一个 Object 对象，同时会将 SparseArray 中的 mGarbage 这个属性设置为 true，这个属性是便于在合适的时候调用自身的 `gc()`方法压缩数组来避免浪费空间。这样可以提高效率，如果将来要添加的key等于删除的key，那么会将要添加的 value 覆盖 DELETE。
- **`gc()。`** SparseArray 中的 `gc()` 方法跟 JVM 的 GC 其实完全没有任何关系。``gc()` 方法的内部实际上就是一个for循环，将 value 不为 DELETE 的键值对往前移动覆盖value 为DELETE的键值对来实现数组的压缩，同时将 mGarbage 置为 false，避免内存的浪费。
- **`put()。`** put 方法是这么一个逻辑，如果通过二分查找 在 mKeys 数组中找到了 key，那么直接覆盖 value 即可。如果没有找到，会拿到与数组中与要添加的 key 最接近的 key 索引，如果这个索引对应的 value 为 DELETE，则直接把新的 value 覆盖 DELET 即可，在这里可以避免数组元素的移动，从而提高了效率。如果 value 不为 DELETE，会判断 mGarbage，如果为 true，则会调用 `gc()`方法压缩数组，之后会找到合适的索引，将索引之后的键值对后移，插入新的键值对，这个过程中可能会触发数组的扩容。

### 图片加载如何避免 OOM

我们知道内存中的 Bitmap 大小的计算公式是：长所占像素 * 宽所占像素 * 每个像素所占内存。想避免 OOM 有两种方法：等比例缩小长宽、减少每个像素所占的内存。

- 等比缩小长宽。我们知道 Bitmap 的创建是通过 BitmapFactory 的工厂方法，`decodeFile()、decodeStream()、decodeByteArray()、decodeResource()`。这些方法中都有一个 Options 类型的参数，这个 Options 是 BitmapFactory 的内部类，存储着 BItmap 的一些信息。Options 中有一个属性：inSampleSize。我们通过修改 inSampleSize 可以缩小图片的长宽，从而减少 BItma p 所占内存。需要注意的是这个 inSampleSize 大小需要是 2 的幂次方，如果小于 1，代码会强制让inSampleSize为1。
- 减少像素所占内存。Options 中有一个属性 inPreferredConfig，默认是 `ARGB_8888`，代表每个像素所占尺寸。我们可以通过将之修改为 `RGB_565` 或者 `ARGB_4444` 来减少一半内存。

### 大图加载

加载高清大图，比如清明上河图，首先屏幕是显示不下的，而且考虑到内存情况，也不可能一次性全部加载到内存。这时候就需要局部加载了，Android中有一个负责局部加载的类：BitmapRegionDecoder。使用方法很简单，通过BitmapRegionDecoder.newInstance()创建对象，之后调用decodeRegion(Rect rect, BitmapFactory.Options options)即可。第一个参数rect是要显示的区域，第二个参数是BitmapFactory中的内部类Options。

### Bitmap计算和内存优化

Bitmap内存占用 ≈ 像素数据总大小 = 图片宽 × 图片高× (设备分辨率/资源目录分辨率)^2 × 每个像素的字节大小（可以详细阐述）

### Bitmap内存优化

图片占用的内存一般会分为运行时占用的运存和存储时本地开销（反映在包大小上），这里我们只关注运行时占用内存的优化。

在上一节中，我们看到对于一张800 * 600 大小的图片，不加任何处理直接解析到内存中，将近占用了17.28M的内存大小。想象一下这样的开销发生在一个图片列表中，内存占用将达到非常夸张的地步。从之前Bitmap占用内存的计算公式来看，减少内存主要可以通过以下几种方式：

1. 使用低色彩的解析模式，如RGB565，减少单个像素的字节大小
2. 资源文件合理放置，高分辨率图片可以放到高分辨率目录下
3. 图片缩小，减少尺寸

**第一种方式**，大约能减少一半的内存开销。Android默认是使用ARGB8888配置来处理色彩，占用4字节，改用RGB565，将只占用2字节，代价是显示的色彩将相对少，适用于对色彩丰富程度要求不高的场景。

**第二种方式**，和图片的具体分辨率有关，建议开发中，高分辨率的图像应该放置到合理的资源目录下，注意到Android默认放置的资源目录是对应于160dpi，目前手机屏幕分辨率越来越高，此处能节省下来的开销也是很可观的。理论上，图片放置的资源目录分辨率越高，其占用内存会越小，但是低分辨率图片会因此被拉伸，显示上出现失真。另一方面，高分辨率图片也意味着其占用的本地储存也变大。

**第三种方式**，理论上根据适用的环境，是可以减少十几倍的内存使用的，它基于这样一个事实：源图片尺寸一般都大于目标需要显示的尺寸，因此可以通过缩放的方式，来减少显示时的图片宽高，从而大大减少占用的内存。

### AsyncTask总结

Android UI是线程不安全的，如果想要在子线程里进行UI操作，就需要借助Android的异步消息处理机制。

不过为了更加方便我们在子线程中更新UI元素，Android从1.5版本就引入了一个AsyncTask类

在Android当中，通常将线程分为两种，一种叫做Main Thread，除了Main Thread之外的线程都可称为Worker Thread

AsyncTask：异步任务，从字面上来说，就是在我们的UI主线程运行的时候，异步的完成一些操作。AsyncTask允许我们的执行一个异步的任务在后台。我们可以将耗时的操作放在异步任务当中来执行，并随时将任务执行的结果返回给我们的UI线程来更新我们的UI控件

我们定义一个类来继承AsyncTask这个类的时候，我们需要为其指定3个泛型参数:

AsyncTask　<Params, Progress, Result>

Params: 这个泛型指定的是我们传递给异步任务执行时的参数的类型

Progress: 这个泛型指定的是我们的异步任务在执行的时候将执行的进度返回给UI线程的参数的类型

Result: 这个泛型指定的异步任务执行完后返回给UI线程的结果的类型

如果都不指定的话，则都将其写成Void

执行一个异步任务的时候，其需要按照下面的4个步骤分别执行:

onPreExecute(): 这个方法是在执行异步任务之前的时候执行，并且是在UI Thread当中执行的，通常我们在这个方法里做一些UI控件的初始化的操作，例如弹出要给ProgressDialog

doInBackground(Params... params): 在onPreExecute()方法执行完之后，会马上执行这个方法，这个方法就是来处理异步任务的方法，Android操作系统会在后台的线程池当中开启一个worker thread来执行我们的这个方法，所以这个方法是在worker thread当中执行的，这个方法执行完之后就可以将我们的执行结果发送给我们的最后一个 onPostExecute 方法，在这个方法里，我们可以从网络当中获取数据等一些耗时的操作

onProgressUpdate(Progess... values): 这个方法也是在UI Thread当中执行的，我们在异步任务执行的时候，有时候需要将执行的进度返回给我们的UI界面，例如下载一张网络图片，我们需要时刻显示其下载的进度，就可以使用这个方法来更新我们的进度。这个方法在调用之前，我们需要在 doInBackground 方法中调用一个 publishProgress(Progress) 的方法来将我们的进度时时刻刻传递给 onProgressUpdate 方法来更新

onPostExecute(Result... result): 当我们的异步任务执行完之后，就会将结果返回给这个方法，这个方法也是在UI Thread当中调用的，我们可以将返回的结果显示在UI控件上

可以在任何时刻来取消我们的异步任务的执行，通过调用 cancel(boolean)方法.

在使用AsyncTask做异步任务的时候必须要遵循的原则：

AsyncTask类必须在UI Thread当中加载，在Android Jelly_Bean版本后这些都是自动完成的

AsyncTask的对象必须在UI Thread当中实例化

execute方法必须在UI Thread当中调用

不要手动的去调用AsyncTask的onPreExecute, doInBackground, publishProgress, onProgressUpdate, onPostExecute方法，这些都是由Android系统自动调用的

AsyncTask任务只能被执行一次

使用的优点:

l 简单,快捷

l 过程可控

使用的缺点:

l 在使用多个异步操作和并需要进行Ui变更时,就变得复杂起来.

### 线性布局和相对布局的效率，约束布局和相对布局的效率

**RelativeLayout**分别对所有子View进行两次measure，横向纵向分别进行一次，这是为什么呢？首先RelativeLayout中子View的排列方式是基于彼此的依赖关系，而这个依赖关系可能和布局中View的顺序并不相同，在确定每个子View的位置的时候，需要先给所有的子View排序一下。又因为RelativeLayout允许A，B 2个子View，横向上B依赖A，纵向上A依赖B。所以需要横向纵向分别进行一次排序测量。 mSortedHorizontalChildren和mSortedVerticalChildren是分别对水平方向的子控件和垂直方向的子控件进行排序后的View数组。

与RelativeLayout相比**LinearLayout**的measure就简单的多，只需判断线性布局是水平布局还是垂直布局即可，然后才进行测量：如果不使用weight属性，LinearLayout会在当前方向上进行一次measure的过程，如果使用weight属性，LinearLayout会避开设置过weight属性的view做第一次measure，完了再对设置过weight属性的view做第二次measure。由此可见，weight属性对性能是有影响的，而且本身有大坑，请注意避让。

**结论**

（1）RelativeLayout会让子View调用2次onMeasure，LinearLayout 在有weight时，也会调用子View 2次onMeasure

（2）RelativeLayout的子View如果高度和RelativeLayout不同，则会引发效率问题，当子View很复杂时，这个问题会更加严重。如果可以，尽量使用padding代替margin。

（3）在不影响层级深度的情况下,使用LinearLayout和FrameLayout而不是RelativeLayout。

（4）提高绘制性能的使用方式

根据上面源码的分析，RelativeLayout将对所有的子View进行两次measure，而LinearLayout在使用weight属性进行布局时也会对子View进行两次measure，如果他们位于整个View树的顶端时并可能进行多层的嵌套时，位于底层的View将会进行大量的measure操作，大大降低程序性能。因此，应尽量将RelativeLayout和LinearLayout置于View树的底层，并减少嵌套

### RecyclerView与ListView 对比：缓存机制

**1. 层级不同：**

RecyclerView比ListView多两级缓存，支持多个离屏ItemView缓存，支持开发者自定义缓存处理逻辑，支持所有RecyclerView共用同一个RecyclerViewPool(缓存池)。

具体来说：

ListView(两级缓存)：

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c875f1e070?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c875f1e070?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

RecyclerView(四级缓存)：

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8760adc75?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8760adc75?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

ListView和RecyclerView缓存机制基本一致：

1). mActiveViews和mAttachedScrap功能相似，意义在于快速重用屏幕上可见的列表项ItemView，而不需要重新createView和bindView；

2). mScrapView和mCachedViews + mReyclerViewPool功能相似，意义在于缓存离开屏幕的ItemView，目的是让即将进入屏幕的ItemView重用.

3). RecyclerView的优势在于a.mCacheViews的使用，可以做到屏幕外的列表项ItemView进入屏幕内时也无须bindView快速重用；b.mRecyclerPool可以供多个RecyclerView共同使用，在特定场景下，如viewpaper+多个列表页下有优势.客观来说，RecyclerView在特定场景下对ListView的缓存机制做了补强和完善。

**2. 缓存不同：**

1). RecyclerView缓存RecyclerView.ViewHolder，抽象可理解为：

View + ViewHolder(避免每次createView时调用findViewById) + flag(标识状态)；

2). ListView缓存View。

# 网络

## http 与 https 的区别？https 是如何工作的？

http 是超文本传输协议，而 https 可以简单理解为安全的 http 协议。https 通过在 http 协议下添加了一层 ssl 协议对数据进行加密从而保证了安全。https 的作用主要有两点：建立安全的信息传输通道，保证数据传输安全；确认网站的真实性。

### http 与 https 的区别主要如下：

- https 需要到 CA 申请证书，很少免费，因而需要一定的费用
- http 是明文传输，安全性低；而 https 在 http 的基础上通过 ssl 加密，安全性高
- 二者的默认端口不一样，http 使用的默认端口是80；https使用的默认端口是 443

### https 的工作流程

提到 https 的话首先要说到加密算法，加密算法分为两类：对称加密和非对称加密。

- **对称加密：** 加密和解密用的都是相同的秘钥，优点是速度快，缺点是安全性低。常见的对称加密算法有 DES、AES 等等。
- **非对称加密：** 非对称加密有一个秘钥对，分为公钥和私钥。一般来说，私钥自己持有，公钥可以公开给对方，优点是安全性比对称加密高，缺点是数据传输效率比对称加密低。采用公钥加密的信息只有对应的私钥可以解密。常见的非对称加密包括RSA等。

在正式的使用场景中一般都是对称加密和非对称加密结合使用，使用非对称加密完成秘钥的传递，然后使用对称秘钥进行数据加密和解密。二者结合既保证了安全性，又提高了数据传输效率。

### https 的具体流程如下：

1. 客户端（通常是浏览器）先向服务器发出加密通信的请求
- 支持的协议版本，比如 TLS 1.0版
- 一个客户端生成的随机数 random1，稍后用于生成"对话密钥"
- 支持的加密方法，比如 RSA 公钥加密
- 支持的压缩方法
1. 服务器收到请求,然后响应
- 确认使用的加密通信协议版本，比如 TLS 1.0版本。如果浏览器与服务器支持的版本不一致，服务器关闭加密通信
- 一个服务器生成的随机数 random2，稍后用于生成"对话密钥"
- 确认使用的加密方法，比如 RSA 公钥加密
- 服务器证书
1. 客户端收到证书之后会首先会进行验证
- 首先验证证书的安全性
- 验证通过之后，客户端会生成一个随机数 pre-master secret，然后使用证书中的公钥进行加密，然后传递给服务器端
1. 服务器收到使用公钥加密的内容，在服务器端使用私钥解密之后获得随机数 pre-master secret，然后根据 radom1、radom2、pre-master secret 通过一定的算法得出一个对称加密的秘钥，作为后面交互过程中使用对称秘钥。同时客户端也会使用 radom1、radom2、pre-master secret，和同样的算法生成对称秘钥。
2. 然后再后续的交互中就使用上一步生成的对称秘钥对传输的内容进行加密和解密。

### http头部的字段以及含义

- **Accept :** 浏览器（或者其他基于HTTP的客户端程序）可以接收的内容类型（Content-types）,例如 `Accept: text/plain`
- **Accept-Charset：**浏览器能识别的字符集，例如 `Accept-Charset: utf-8`
- **Accept-Encoding：**浏览器可以处理的编码方式，注意这里的编码方式有别于字符集，这里的编码方式通常指gzip,deflate等。例如 `Accept-Encoding: gzip, deflate`
- **Accept-Language：**浏览器接收的语言，其实也就是用户在什么语言地区，例如简体中文的就是 `Accept-Language: zh-CN`
- **Authorization：**在HTTP中，服务器可以对一些资源进行认证保护，如果你要访问这些资源，就要提供用户名和密码，这个用户名和密码就是在Authorization头中附带的，格式是“username:password”字符串的base64编码
- **Cache-Control：**这个指令在request和response中都有，用来指示缓存系统（服务器上的，或者浏览器上的）应该怎样处理缓存，因为这个头域比较重要，特别是希望使用缓　存改善性能的时候
- **Connection：**告诉服务器这个user agent（通常就是浏览器）想要使用怎样的连接方式。值有keep-alive和close。http1.1默认是keep-alive。keep-alive就是浏览器和服务器　的通信连接会被持续保存，不会马上关闭，而close就会在response后马上关闭。但这里要注意一点，我们说HTTP是无状态的，跟这个是否keep-alive没有关系，不要认为keep-alive是对HTTP无状态的特性的改进。
- **Cookie：**浏览器向服务器发送请求时发送cookie，或者服务器向浏览器附加cookie，就是将cookie附近在这里的。例如：`Cookie:user=admin`
- **Content-Length：**一个请求的请求体的内存长度，单位为字节(byte)。请求体是指在HTTP头结束后，两个CR-LF字符组之后的内容，常见的有POST提交的表单数据，这个Content-Length并不包含请求行和HTTP头的数据长度。
- **Content-MD5：**使用base64进行了编码的请求体的MD5校验和。例如：`Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==`
- **Content-Type：**请求体中的内容的mime类型。通常只会用在POST和PUT方法的请求中。例如：`Content-Type: application/x-www-form-urlencoded`
- **Date：**发送请求时的GMT时间。例如：`Date: Tue, 15 Nov 1994 08:12:31 GMT`
- **From：**发送这个请求的用户的email地址。例如：`From: user@example.com`
- **Host：**被服务器的域名或IP地址，如果不是通用端口，还包含该端口号，例如：`Host: www.some.com:182`
- **Proxy-Authorization：**连接到某个代理时使用的身份认证信息，跟Authorization头差不多。例如：`Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==`
- **User-Agent：**通常就是用户的浏览器相关信息。例如：`User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:12.0) Gecko/20100101 Firefox/12.0`
- **Warning：**记录一些警告信息。

# Dart

### 1. Dart 当中的 「..」表示什么意思？

Dart 当中的 「..」意思是 「级联操作符」，为了方便配置而使用。「..」和「.」不同的是 调用「..」后返回的相当于是 this，而「.」返回的则是该方法返回的值 。

### 变量声明总结

1. var: 如果没有初始值，可以变成任何类型
2. dynamic:动态任意类型，编译阶段不检查类型
3. Object 动态任意类型，编译阶段检查类型区别：
4. 唯一区别 var 如果有初始值，类型被是锁定

### final 和 const 总结

共同点

- 声明的类型可省略
- 初始化后不能再赋值
- 不能和 var 同时使用

区别（需要注意的地方）

- 类级别常量，使用 static ，const 。
- const 可使用其他 const 常量的值来初始化其值
- const 可使用其他 const 常量的值来初始化其值
- 可以更改非 final、非 const 变量的值，即使曾经具有 const 值
- const 导致的不可变性是可传递的
- 相同的 const 常量不会在内存中重复创建
- const 需要是编译时常量

### 操作符

### 后缀操作

- 条件成员访问 和 . 类似，但是左边的操作对象不能为 null，例如 foo?.bar 如果 foo 为 null 则返回 null，否则返回 bar 成员

### 取商操作符

- 被除数 ÷ 除数 = 商 ... 余数，A ~/ B = C，这个C就是商。相当于Java里的 /

### 类型判定操作符

- as、is、is! 在运行时判定对象类型

### 条件表达式

- 三目运算符 condition ? expr1 : expr2

### 流程控制语句

if else

for , forEach , for-in

### 2. Dart 的作用域

Dart 没有 「public」「private」等关键字，默认就是公开的，私有变量使用 下划线 _开头。“_”的限制范围并不是类访问级别的，而是库访问级别。

### 3. Dart 是不是单线程模型？是如何运行的？

Dart 是单线程模型，运行的的流程如下图。

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c876c2860c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c876c2860c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c87e7d9815?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c87e7d9815?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

简单来说，Dart 在单线程中是以消息循环机制来运行的，包含两个任务队列，一个是“微任务队列” microtask queue，另一个叫做“事件队列” event queue。

当Flutter应用启动后，消息循环机制便启动了。首先会按照先进先出的顺序逐个执行微任务队列中的任务，当所有微任务队列执行完后便开始执行事件队列中的任务，事件任务执行完毕后再去执行微任务，如此循环往复，生生不息。

### 4. Dart 是如何实现多任务并行的？

Dart 是单线程的，不存在多线程，那如何进行多任务并行的呢？其实，Dart的多线程和前端的多线程有很多的相似之处。Flutter的多线程主要依赖Dart的并发编程、异步和事件驱动机制。

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c87fdeecce?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c87fdeecce?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

简单的说，在Dart中，一个Isolate对象其实就是一个isolate执行环境的引用，一般来说我们都是通过当前的isolate去控制其他的isolate完成彼此之间的交互，而当我们想要创建一个新的Isolate可以使用Isolate.spawn方法获取返回的一个新的isolate对象，两个isolate之间使用SendPort相互发送消息，而isolate中也存在了一个与之对应的ReceivePort接受消息用来处理，但是我们需要注意的是，ReceivePort和SendPort在每个isolate都有一对，只有同一个isolate中的ReceivePort才能接受到当前类的SendPort发送的消息并且处理。

### future、async 和 await

Future 是异步编程的解决方案，Future 是基于观察者模式的，它有 3 种状态：pending（进行中）、fulfilled（已成功）和 rejected（已失败）可以用 then 方法指定成功状态的回调函数 then 方法还可以接受一个可选命名参数，参数的名称是 onError，即失败状态的回调函数 Future 实例的函数中抛出了异常，被 onError 回调函数捕获到，并且可以看出 then 方法返回的还是一个 Future 对象，所以其实我们还可以利用 Future 对象的 cathError 进行链式调用从而捕获异常

在Dart1.9中加入了`async`和`await`关键字，有了这两个关键字，我们可以更简洁的编写异步代码，而不需要调用`Future`相关的API

将 `async` 关键字作为方法声明的后缀时，具有如下意义

- 被修饰的方法会将一个 `Future` 对象作为返回值
- 该方法会**同步**执行其中的方法的代码直到**第一个 await 关键字**，然后它暂停该方法其他部分的执行；
- 一旦由 **await** 关键字引用的 **Future** 任务执行完成，**await**的下一行代码将立即执行。

**async** 不是并行执行，它是遵循Dart **事件循环**规则来执行的，它仅仅是一个语法糖，简化`Future API`的使用。

### Dart异步编程中的 Stream数据流？

在Dart中，Stream 和 Future 一样，都是用来处理异步编程的工具。它们的区别在于，Stream 可以接收多个异步结果，而Future 只有一个。

Stream 的创建可以使用 Stream.fromFuture，也可以使用 StreamController 来创建和控制。还有一个注意点是：普通的 Stream 只可以有一个订阅者，如果想要多订阅的话，要使用 asBroadcastStream()。

### Stream 有哪两种订阅模式？分别是怎么调用的

Stream有两种订阅模式：单订阅(single) 和 多订阅（broadcast）。单订阅就是只能有一个订阅者，而广播是可以有多个订阅者。这就有点类似于消息服务（Message Service）的处理模式。单订阅类似于点对点，在订阅者出现之前会持有数据，在订阅者出现之后就才转交给它。而广播类似于发布订阅模式，可以同时有多个订阅者，当有数据时就会传递给所有的订阅者，而不管当前是否已有订阅者存在。

Stream 默认处于单订阅模式，所以同一个 stream 上的 listen 和其它大多数方法只能调用一次，调用第二次就会报错。但 Stream 可以通过 transform() 方法（返回另一个 Stream）进行连续调用。通过 Stream.asBroadcastStream() 可以将一个单订阅模式的 Stream 转换成一个多订阅模式的 Stream，isBroadcast 属性可以判断当前 Stream 所处的模式。

### mixin机制

mixin 是Dart 2.1 加入的特性，以前版本通常使用abstract class代替。简单来说，mixin是为了解决继承方面的问题而引入的机制，Dart为了支持多重继承，引入了mixin关键字，它最大的特殊处在于：mixin定义的类不能有构造方法，这样可以避免继承多个类而产生的父类构造方法冲突。

mixins的对象是类，mixins绝不是继承，也不是接口，而是一种全新的特性，可以mixins多个类，mixins的使用需要满足一定条件。

### JIT 与 AOT

借助于先进的工具链和编译器，Dart 是少数同时支持 JIT（Just In Time，即时编译）和 AOT（Ahead of Time，运行前编译）的语言之一。那，到底什么是 JIT 和 AOT 呢？语言在运行之前通常都需要编译，JIT 和 AOT 则是最常见的两种编译模式。JIT 在运行时即时编译，在开发周期中使用，可以动态下发和执行代码，开发测试效率高，但运行速度和执行性能则会因为运行时即时编译受到影响。AOT 即提前编译，可以生成被直接执行的二进制代码，运行速度快、执行性能表现好，但每次执行前都需要提前编译，开发测试效率低。

### 内存分配与垃圾回收

Dart VM 的内存分配策略比较简单，创建对象时只需要在堆上移动指针，内存增长始终是线性的，省去了查找可用内存的过程。在 Dart 中，并发是通过 Isolate 实现的。Isolate 是类似于线程但不共享内存，独立运行的 worker。这样的机制，就可以让 Dart 实现无锁的快速分配。Dart 的垃圾回收，则是采用了多生代算法。新生代在回收内存时采用“半空间”机制，触发垃圾回收时，Dart 会将当前半空间中的“活跃”对象拷贝到备用空间，然后整体释放当前空间的所有内存。回收过程中，Dart 只需要操作少量的“活跃”对象，没有引用的大量“死亡”对象则被忽略，这样的回收机制很适合 Flutter 框架中大量 Widget 销毁重建的场景。

# Flutter

### 简单介绍下Flutter框架，以及它的优缺点？

Flutter是Google推出的一套开源跨平台UI框架，可以快速地在Android、iOS和Web平台上构建高质量的原生用户界面。同时，Flutter还是Google新研发的Fuchsia操作系统的默认开发套件。在全世界，Flutter正在被越来越多的开发者和组织使用，并且Flutter是完全免费、开源的。Flutter采用现代响应式框架构建，其中心思想是使用组件来构建应用的UI。当组件的状态发生改变时，组件会重构它的描述，Flutter会对比之前的描述，以确定底层渲染树从当前状态转换到下一个状态所需要的最小更改。

**优点**

- 热重载（Hot Reload），利用Android Studio直接一个ctrl+s就可以保存并重载，模拟器立马就可以看见效果，相比原生冗长的编译过程强很多；
- 一切皆为Widget的理念，对于Flutter来说，手机应用里的所有东西都是Widget，通过可组合的空间集合、丰富的动画库以及分层课扩展的架构实现了富有感染力的灵活界面设计；
- 借助可移植的GPU加速的渲染引擎以及高性能本地代码运行时以达到跨平台设备的高质量用户体验。简单来说就是：最终结果就是利用Flutter构建的应用在运行效率上会和原生应用差不多。

**缺点**

- 不支持热更新；
- 三方库有限，需要自己造轮子；
- Dart语言编写，增加了学习难度，并且学习了Dart之后无其他用处，相比JS和Java来说。

### 介绍下Flutter的理念架构

其实也就是下面这张图。

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c884fbc3f3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c884fbc3f3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

由上图可知，Flutter框架自下而上分为Embedder、Engine和Framework三层。其中，Embedder是操作系统适配层，实现了渲染 Surface设置，线程设置，以及平台插件等平台相关特性的适配；Engine层负责图形绘制、文字排版和提供Dart运行时，Engine层具有独立虚拟机，正是由于它的存在，Flutter程序才能运行在不同的平台上，实现跨平台运行；Framework层则是使用Dart编写的一套基础视图库，包含了动画、图形绘制和手势识别等功能，是使用频率最高的一层。

### Flutter的FrameWork层和Engine层，以及它们的作用

Flutter的FrameWork层是用Drat编写的框架（SDK），它实现了一套基础库，包含Material（Android风格UI）和Cupertino（iOS风格）的UI界面，下面是通用的Widgets（组件），之后是一些动画、绘制、渲染、手势库等。这个纯 Dart实现的 SDK被封装为了一个叫作 dart:ui的 Dart库。我们在使用 Flutter写 App的时候，直接导入这个库即可使用组件等功能。

Flutter的Engine层是Skia 2D的绘图引擎库，其前身是一个向量绘图软件，Chrome和 Android均采用 Skia作为绘图引擎。Skia提供了非常友好的 API，并且在图形转换、文字渲染、位图渲染方面都提供了友好、高效的表现。Skia是跨平台的，所以可以被嵌入到 Flutter的 iOS SDK中，而不用去研究 iOS闭源的 Core Graphics / Core Animation。Android自带了 Skia，所以 Flutter Android SDK要比 iOS SDK小很多。

### 介绍下Widget、State、Context 概念

- **Widget**：在Flutter中，几乎所有东西都是Widget。将一个Widget想象为一个可视化的组件（或与应用可视化方面交互的组件），当你需要构建与布局直接或间接相关的任何内容时，你正在使用Widget。
- **Widget树**：Widget以树结构进行组织。包含其他Widget的widget被称为父Widget(或widget容器)。包含在父widget中的widget被称为子Widget。
- **Context**：仅仅是已创建的所有Widget树结构中的某个Widget的位置引用。简而言之，将context作为widget树的一部分，其中context所对应的widget被添加到此树中。一个context只从属于一个widget，它和widget一样是链接在一起的，并且会形成一个context树。
- **State**：定义了StatefulWidget实例的行为，它包含了用于”交互/干预“Widget信息的行为和布局。应用于State的任何更改都会强制重建Widget。

### StatelessWidget和StatefulWidget两种状态组件类

- **StatelessWidget**: 一旦创建就不关心任何变化，在下次构建之前都不会改变。它们除了依赖于自身的配置信息（在父节点构建时提供）外不再依赖于任何其他信息。比如典型的Text、Row、Column、Container等，都是StatelessWidget。它的生命周期相当简单：初始化、通过build()渲染。
- **StatefulWidget**: 在生命周期内，该类Widget所持有的数据可能会发生变化，这样的数据被称为State，这些拥有动态内部数据的Widget被称为StatefulWidget。比如复选框、Button等。State会与Context相关联，并且此关联是永久性的，State对象将永远不会改变其Context，即使可以在树结构周围移动，也仍将与该context相关联。当state与context关联时，state被视为已挂载。StatefulWidget由两部分组成，在初始化时必须要在createState()时初始化一个与之相关的State对象。

### StatefulWidget 的生命周期

Flutter的Widget分为StatelessWidget和StatefulWidget两种。其中，StatelessWidget是无状态的，StatefulWidget是有状态的，因此实际使用时，更多的是StatefulWidget。StatefulWidget的生命周期如下图

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c89ccdfe75?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c89ccdfe75?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- **initState()**：Widget 初始化当前 State，在当前方法中是不能获取到 Context 的，如想获取，可以试试 Future.delayed()
- **didChangeDependencies()**：在 initState() 后调用，State对象依赖关系发生变化的时候也会调用。
- **deactivate()**：当 State 被暂时从视图树中移除时会调用这个方法，页面切换时也会调用该方法，和Android里的 onPause 差不多。
- **dispose()**：Widget 销毁时调用。
- **didUpdateWidget**：Widget 状态发生变化的时候调用。

### Widgets、RenderObjects 和 Elements的关系

首先看一下这几个对象的含义及作用。

- **Widget** ：仅用于存储渲染所需要的信息。
- **RenderObject** ：负责管理布局、绘制等操作。
- **Element** ：才是这颗巨大的控件树上的实体。

Widget会被inflate（填充）到Element，并由Element管理底层渲染树。Widget并不会直接管理状态及渲染,而是通过State这个对象来管理状态。Flutter创建Element的可见树，相对于Widget来说，是可变的，通常界面开发中，我们不用直接操作Element,而是由框架层实现内部逻辑。就如一个UI视图树中，可能包含有多个TextWidget(Widget被使用多次)，但是放在内部视图树的视角，这些TextWidget都是填充到一个个独立的Element中。Element会持有renderObject和widget的实例。记住，Widget 只是一个配置，RenderObject 负责管理布局、绘制等操作。

在第一次创建 Widget 的时候，会对应创建一个 Element， 然后将该元素插入树中。如果之后 Widget 发生了变化，则将其与旧的 Widget 进行比较，并且相应地更新 Element。重要的是，Element 不会被重建，只是更新而已。

### 什么是状态管理，你了解哪些状态管理框架？

Flutter中的状态和前端React中的状态概念是一致的。React框架的核心思想是组件化，应用由组件搭建而成，组件最重要的概念就是状态，状态是一个组件的UI数据模型，是组件渲染时的数据依据。

Flutter的状态可以分为全局状态和局部状态两种。常用的状态管理有ScopedModel、BLoC、Redux / FishRedux和Provider。详细使用情况和差异可以自行了解。

### Flutter的绘制流程

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8a01e1409?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8a01e1409?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Flutter只关心向 GPU提供视图数据，GPU的 VSync信号同步到 UI线程，UI线程使用 Dart来构建抽象的视图结构，这份数据结构在 GPU线程进行图层合成，视图数据提供给 Skia引擎渲染为 GPU数据，这些数据通过 OpenGL或者 Vulkan提供给 GPU。

### Flutter的线程管理模型

默认情况下，Flutter Engine层会创建一个Isolate，并且Dart代码默认就运行在这个主Isolate上。必要时可以使用spawnUri和spawn两种方式来创建新的Isolate，在Flutter中，新创建的Isolate由Flutter进行统一的管理。

事实上，Flutter Engine自己不创建和管理线程，Flutter Engine线程的创建和管理是Embeder负责的，Embeder指的是将引擎移植到平台的中间层代码，Flutter Engine层的架构示意图如下图所示。

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8a1ccb618?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8a1ccb618?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

在Flutter的架构中，Embeder提供四个Task Runner，分别是Platform Task Runner、UI Task Runner Thread、GPU Task Runner和IO Task Runner，每个Task Runner负责不同的任务，Flutter Engine不在乎Task Runner运行在哪个线程，但是它需要线程在整个生命周期里面保持稳定

### Flutter 是如何与原生Android、iOS进行通信的？

Flutter 通过 PlatformChannel 与原生进行交互，其中 PlatformChannel 分为三种：

- **BasicMessageChannel** ：用于传递字符串和半结构化的信息。
- **MethodChannel** ：用于传递方法调用（method invocation）。
- **EventChannel** : 用于数据流（event streams）的通信。

同时 Platform Channel 并非是线程安全的 ，更多详细可查阅闲鱼技术的 《深入理解Flutter Platform Channel》

### 简述Flutter 的热重载

Flutter 的热重载是基于 JIT 编译模式的代码增量同步。由于 JIT 属于动态编译，能够将 Dart 代码编译成生成中间代码，让 Dart VM 在运行时解释执行，因此可以通过动态更新中间代码实现增量同步。

热重载的流程可以分为 5 步，包括：扫描工程改动、增量编译、推送更新、代码合并、Widget 重建。Flutter 在接收到代码变更后，并不会让 App 重新启动执行，而只会触发 Widget 树的重新绘制，因此可以保持改动前的状态，大大缩短了从代码修改到看到修改产生的变化之间所需要的时间。

另一方面，由于涉及到状态的保存与恢复，涉及状态兼容与状态初始化的场景，热重载是无法支持的，如改动前后 Widget 状态无法兼容、全局变量与静态属性的更改、main 方法里的更改、initState 方法里的更改、枚举和泛型的更改等。

可以发现，热重载提高了调试 UI 的效率，非常适合写界面样式这样需要反复查看修改效果的场景。但由于其状态保存的机制所限，热重载本身也有一些无法支持的边界。

### Flutter 是怎么运转的？

与用于构建移动应用程序的其他大多数框架不同，Flutter 是重写了一整套包括底层渲染逻辑和上层开发语言的完整解决方案。这样不仅可以保证视图渲染在 Android 和 iOS 上的高度一致性（即高保真），在代码执行效率和渲染性能上也可以媲美原生 App 的体验（即高性能）。这，就是

### Flutter 和其他跨平台方案的本质区别：

React Native 之类的框架，只是通过 JavaScript 虚拟机扩展调用系统组件，由 Android 和 iOS 系统进行组件的渲染；

Flutter 则是自己完成了组件渲染的闭环。那么，Flutter 是怎么完成组件渲染的呢？这需要从图像显示的基本原理说起。在计算机系统中，图像的显示需要 CPU、GPU 和显示器一起配合完成：CPU 负责图像数据计算，GPU 负责图像数据渲染，而显示器则负责最终图像显示。CPU 把计算好的、需要显示的内容交给 GPU，由 GPU 完成渲染后放入帧缓冲区，随后视频控制器根据垂直同步信号（VSync）以每秒 60 次的速度，从帧缓冲区读取帧数据交由显示器完成图像显示。操作系统在呈现图像时遵循了这种机制，而 Flutter 作为跨平台开发框架也采用了这种底层方案。下面有一张更为详尽的示意图来解释 Flutter 的绘制原理。

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8b96b04a4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8b96b04a4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Flutter 绘制原理可以看到，Flutter 关注如何尽可能快地在两个硬件时钟的 VSync 信号之间计算并合成视图数据，然后通过 Skia 交给 GPU 渲染：UI 线程使用 Dart 来构建视图结构数据，这些数据会在 GPU 线程进行图层合成，随后交给 Skia 引擎加工成 GPU 数据，而这些数据会通过 OpenGL 最终提供给 GPU 渲染。

## 架构

### 组件化

组件化又叫模块化，即基于可重用的目的，将一个大型软件系统（App）按照关注点分离的方式，拆分成多个独立的组件或模块。每个独立的组件都是一个单独的系统，可以单独维护、升级甚至直接替换，也可以依赖于别的独立组件，只要组件提供的功能不发生变化，就不会影响其他组件和软件系统的整体功能。

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8bc9406f7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8bc9406f7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

可以看到，组件化的中心思想是将独立的功能进行拆分，而在拆分粒度上，组件化的约束则较为松散。一个独立的组件可以是一个软件包（Package）、页面、UI 控件，甚至可能是封装了一些函数的模块。组件的粒度可大可小，那我们如何才能做好组件的封装重用呢？哪些代码应该被放到一个组件中？这里有一些基本原则，包括单一性原则、抽象化原则、稳定性原则和自完备性原则。接下来，我们先看看这些原则具体是什么意思。

**单一性原则**指的是，每个组件仅提供一个功能。分而治之是组件化的中心思想，每个组件都有自己固定的职责和清晰的边界，专注地做一件事儿，这样这个组件才能良性发展。一个反例是 Common 或 Util 组件，这类组件往往是因为在开发中出现了定义不明确、归属边界不清晰的代码：“哎呀，这段代码放哪儿好像都不合适，那就放 Common（Util）吧”。久而久之，这类组件就变成了无人问津的垃圾堆。所以，再遇到不知道该放哪儿的代码时，就需要重新思考组件的设计和职责了。

**抽象化原则**指的是，组件提供的功能抽象应该尽量稳定，具有高复用度。而稳定的直观表现就是对外暴露的接口很少发生变化，要做到这一点，需要我们提升对功能的抽象总结能力，在组件封装时做好功能抽象和接口设计，将所有可能发生变化的因子都在组件内部做好适配，不要暴露给它的调用方。

**稳定性原则**指的是，不要让稳定的组件依赖不稳定的组件。比如组件 1 依赖了组件 5，如果组件 1 很稳定，但是组件 5 经常变化，那么组件 1 也就会变得不稳定了，需要经常适配。如果组件 5 里确实有组件 1 不可或缺的代码，我们可以考虑把这段代码拆出来单独做成一个新的组件 X，或是直接在组件 1 中拷贝一份依赖的代码。

**自完备性**，即组件需要尽可能地做到自给自足，尽量减少对其他底层组件的依赖，达到代码可复用的目的。比如，组件 1 只是依赖某个大组件 5 中的某个方法，这时更好的处理方法是，剥离掉组件 1 对组件 5 的依赖，直接把这个方法拷贝到组件 1 中。这样一来组件 1 就能够更好地应对后续的外部变更了。

在理解了组件化的基本原则之后，我们再来看看组件化的具体实施步骤，即**剥离基础功能、抽象业务模块和最小化服务能力**。首先，我们需要剥离应用中与业务无关的基础功能，比如网络请求、组件中间件、第三方库封装、UI 组件等，将它们封装为独立的基础库；然后，我们在项目里用 pub 进行管理。如果是第三方库，考虑到后续的维护适配成本，我们最好再封装一层，使项目不直接依赖外部代码，方便后续更新或替换。基础功能已经封装成了定义更清晰的组件，接下来我们就可以按照业务维度，比如首页、详情页、搜索页等，去拆分独立的模块了。拆分的粒度可以先粗后细，只要能将大体划分清晰的业务组件进行拆分，后续就可以通过分布迭代、局部微调，最终实现整个业务项目的组件化。在业务组件和基础组件都完成拆分封装后，应用的组件化架构就基本成型了，最后就可以按照刚才我们说的 4 个原则，去修正各个组件向下的依赖，以及最小化对外暴露的能力了。

### 平台化

从组件的定义可以看到，组件是个松散的广义概念，其规模取决于我们封装的功能维度大小，而各个组件之间的关系也仅靠依赖去维持。如果组件之间的依赖关系比较复杂，就会在一定程度上造成功能耦合现象。

如下所示的组件示意图中，组件 2 和组件 3 同时被多个业务组件和基础功能组件直接引用，甚至组件 2 和组件 5、组件 3 和组件 4 之间还存在着循环依赖的情况。一旦这些组件的内部实现和外部接口发生变化，整个 App 就会陷入不稳定的状态，即所谓牵一发而动全身

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8c5adbc20?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8c5adbc20?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

平台化是组件化的升级，即在组件化的基础上，对它们提供的功能进行分类，统一分层划分，增加依赖治理的概念。为了对这些功能单元在概念上进行更为统一的分类，我们按照四象限分析法，把应用程序的组件按照业务和 UI 分解为 4 个维度，来分析组件可以分为哪几类。

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8cbef1ada?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8cbef1ada?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

可以看出，经过业务与 UI 的分解之后，这些组件可以分为 4 类：

具备 UI 属性的独立业务模块；

不具备 UI 属性的基础业务功能；

不具备业务属性的 UI 控件

不具备业务属性的基础功能

按照自身定义，这 4 类组件其实隐含着分层依赖的关系。比如，处于业务模块中的首页，依赖位于基础业务模块中的账号功能；再比如，位于 UI 控件模块中的轮播卡片，依赖位于基础功能模块中的存储管理等功能。我们将它们按照依赖的先后顺序从上到下进行划分，就是一个完整的 App 了

[https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8cc6403ed?imageView2/0/w/1280/h/960/format/webp/ignore-error/1](https://user-gold-cdn.xitu.io/2020/4/24/171ab6c8cc6403ed?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

可以看到，平台化与组件化最大的差异在于增加了分层的概念，每一层的功能均基于同层和下层的功能之上，这使得各个组件之间既保持了独立性，同时也具有一定的弹性，在不越界的情况下按照功能划分各司其职。

**与组件化更关注组件的独立性相比，平台化更关注的是组件之间关系的合理性，而这也是在设计平台化架构时需要重点考虑的单向依赖原则。**

所谓**单向依赖原则**，指的是组件依赖的顺序应该按照应用架构的层数从上到下依赖，不要出现下层模块依赖上层模块这样循环依赖的现象。这样可以最大限度地避免复杂的耦合，减少组件化时的困难。如果我们每个组件都只是单向依赖其他组件，各个组件之间的关系都是清晰的，代码解耦也就会变得非常轻松了。平台化强调依赖的顺序性，除了不允许出现下层组件依赖上层组件的情况，跨层组件和同层组件之间的依赖关系也应当严格控制，因为这样的依赖关系往往会带来架构设计上的混乱。

**如果下层组件确实需要调用上层组件的代码怎么办？**

这时，我们可以采用增加中间层的方式，比如 Event Bus、Provider 或 Router，以中间层转发的形式实现信息同步。比如，位于第 4 层的网络引擎中，会针对特定的错误码跳转到位于第 1 层的统一错误页，这时我们就可以利用 Router 提供的命名路由跳转，在不感知错误页的实现情况下来完成。又比如，位于第 2 层的账号组件中，会在用户登入登出时主动刷新位于第 1 层的首页和我的页面，这时我们就可以利用 Event Bus 来触发账号切换事件，在不需要获取页面实例的情况下通知它们更新界面。