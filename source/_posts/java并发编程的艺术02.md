---
title: java并发编程的艺术02
date: 2018-10-11 17:38:38
categories: 并发
---

> 以下是《java并发编程的艺术》一书的读书笔记第二部分。

<!-- more -->

#### Java内存模型 

在Java中，所有实例域、静态域和数组元素都存储在堆内存中。局部变量（Local Variables），方法定义参数（Java语言规范称之为Formal Method Parameters）和异常处理器参数（ExceptionHandler Parameters）不会在线程之间共享。

Java线程之间的通信由Java内存模型（本文简称为JMM）控制， 线程之间的共享变量存储在主内存（Main Memory）中，每个线程都有一个私有的本地内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化。

![jmm](\images\java并发编程的艺术02\jmm.jpg)

为了提高性能，编译器和处理器常常会对指令做重排序。重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段。重排序可能会导致多线程程序出现内存可见性问题。

与程序员密切相关的happens-before规则如下。

- 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。

- 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。

- volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。

- 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。

  > 两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second）。

```
double pi = 3.14; // A
double r = 1.0; // B
double area = pi * r * r; // C
```

上面计算圆的面积的示例代码存在3个happensbefore关系。
1）A happens-before B。
2）B happens-before C。
3）A happens-before C。
这里的第3个happens-before关系，是根据happens-before的传递性推导出来的。这里A happens-before B，但实际执行时B却可以排在A之前执行（看上面的重排序后的执行顺序）。如果A happens-before B，JMM并不要求A一定要在B之前执行。JMM仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前。这里操作A的执行结果不需要对操作B可见；而且重排序操作A和操作B后的执行结果，与操作A和操作B按happens-before顺序执行的结果一致。在这种情况下，JMM会认为这种重排序并不非法（not illegal），JMM允许这种重排序。 

#### 重排序问题

数据依赖：如果两个操作访问同一个变量，且这两个操作中有一个为写操作，此时这两个操作之间就存在数据依赖性。数据依赖分为 写后读、写后写、读后写 3 种。写后读 代表了 写一个变量之后再读这个变量，其他的也类似。对于这 3 种情况，只要重排序两个操作的执行顺序，程序的执行结果就会被改变。编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序，但不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑。

控制依赖：as-if-serial语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。编译器、runtime和处理器都必须遵守as-if-serial语义。asif-serial语义使单线程程序员无需担心重排序会干扰他们，也无需担心内存可见性问题。

有了以上两个基础来看下面的问题：

```
class ReorderExample {
    int a = 0;
    boolean flag = false;
    public void writer() {
        a = 1; // 1
        flag = true; // 2
    } 
    public void reader() {
        if (flag) { // 3
        int i = a * a; // 4
        ……
        }
    }
}
```

这里假设有两个线程A和B，A首先执行writer()方法，随后B线程接着执行reader()方法。线程B在执行操作4时，能否看到线程A在操作1对共享变量a的写入呢？

**答案是：不一定能看到。**

由于操作1和操作2没有数据依赖关系，编译器和处理器可以对这两个操作重排序；同样，操作3和操作4没有数据依赖关系但存在控制依赖，单线程情况下，对存在控制依赖的操作重排序，不会改变执行结果（这也是as-if-serial 语义允许对存在控制依赖的操作做重排序的原因）编译器和处理器也可以对这两个操作重排序。

操作1和操作2可以重排序比较容易理解，操作3和操作4的重排序则可以参考下图进行理解：

![控制依赖重排序](\images\java并发编程的艺术02\控制依赖重排序.jpg)

其中，当代码中存在控制依赖性时，会影响指令序列执行的并行度。为此编译器和处理器会采用猜测（Speculation）执行来克服控制相关性对并行度的影响。以处理器的猜测执行为例，执行线程B的处理器可以提前读取并计算a*a，然后把计算结果临时保存到一个名为重排序缓冲（Reorder Buffer，ROB）的硬件缓存中。当操作3的条件判断为真时，就把该计算结果写入变量i中。这样一来便产生了操作3与操作4的重排序。

由此可以得知，未同步程序在JMM中不但整体的执行顺序是无序的，而且所有线程看到的操作执行顺序也可能不一致。比如，在当前线程把写过的数据缓存在本地内存中，在没有刷新到主内存之前，这个写操作仅对当前线程可见；从其他线程的角度来观察，会认为这个写操作根本没有被当前线程执行。

现在对上面的程序进行改造如下

```
class SynchronizedExample {
    int a = 0;
    boolean flag = false;
    public synchronized void writer() { // 获取锁
        a = 1;
        flag = true;
    } // 释放锁
    public synchronized void reader() { // 获取锁
        if (flag) {
            int i = a;
            ……
        } // 释放锁
    }
}
```

添加同步锁后多线程可以正确执行。原因是在JMM中，临界区内的代码可以重排序，但JMM会在退出临界区和进入临界区这两个关键时间点做一些特别处理，使得线程在这两个时间点具有与顺序一致性。所以虽然线程A在临界区内做了重排序，但线程B由于没有锁（或者说 monitor）根本无法“观察”到线程A在临界区内的重排序。这种重排序既提高了执行效率，又没有改变程序的执行结果。

![jmm与顺序一致模型对比](\images\java并发编程的艺术02\jmm与顺序一致模型对比.jpg)

由此可知，JMM在具体实现上的基本方针为：在不改变（正确同步的）程序执行结果的前提下，尽可能地为编译器和处理器的优化让步。

#### volatile的内存语义

为了理解 volatile ，看下面的程序：	

```
class VolatileFeaturesExample {
    volatile long vl = 0L; // 使用volatile声明64位的long型变量
    public void set(long l) {
    	vl = l; // 单个volatile变量的写
    } 
    public void getAndIncrement () {
    	vl++; // 复合（多个）volatile变量的读/写
    } 
    public long get() {
    	return vl; // 单个volatile变量的读
    }
}
```

假设有多个线程分别调用上面程序的3个方法，这个程序在语义上和下面程序等价。

```
class VolatileFeaturesExample {
    long vl = 0L; // 64位的long型普通变量
    public synchronized void set(long l) { // 对单个的普通变量的写用同一个锁同步
    	vl = l;
    } 
    public void getAndIncrement () { // 普通方法调用
        long temp = get(); // 调用已同步的读方法
        temp += 1L; // 普通写操作
        set(temp); // 调用已同步的写方法
    } 
    public synchronized long get() { // 对单个的普通变量的读用同一个锁同步
    	return vl;
    }
}
```

一个volatile变量的单个读/写操作，与一个普通变量的读/写操作都是使用同一个锁来同步，它们之间的执行效果相同。锁的happens-before规则保证释放锁和获取锁的两个线程之间的内存可见性，这意味着对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。但需要注意，似于volatile++这种复合操作不具有原子性。

所以，volatile有如下语义：

- 线程A写一个volatile变量，实质上是线程A向接下来将要读这个volatile变量的某个线程发出了（其对共享变量所做修改的）消息。
- 线程B读一个volatile变量，实质上是线程B接收了之前某个线程发出的（在写这个volatile变量之前对共享变量所做修改的）消息。
- 线程A写一个volatile变量，随后线程B读这个volatile变量，这个过程实质上是线程A通过主内存向线程B发送消息。

前文提到过重排序分为编译器重排序和处理器重排序。而为了实现volatile内存语义，JMM会分别限制这两种类型的重排序类型。下表是JMM针对编译器制定的volatile重排序规则表。

![volatile重排序规则](\images\java并发编程的艺术02\volatile重排序规则.jpg)

由此我们可知：

- 当第二个操作是volatile写时，不管第一个操作是什么，都不能重排序。这个规则确保volatile写之前的操作不会被编译器重排序到volatile写之后。
- 当第一个操作是volatile读时，不管第二个操作是什么，都不能重排序。这个规则确保volatile读之后的操作不会被编译器重排序到volatile读之前。
- 当第一个操作是volatile写，第二个操作是volatile读时，不能重排序。

这样一来就保证了 volatile 的语义。

总结：volatile仅仅保证对单个volatile变量的读/写具有原子性，而锁的互斥执行的特性可以确保对整个临界区代码的执行具有原子性。在功能上，锁比volatile更强大；在可伸缩性和执行性能上，volatile更有优势。

#### 锁的内存语义

锁除了让临界区互斥执行外，还可以让释放锁的线程向获取同一个锁的线程发送消息。线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中。以下面的MonitorExample程序为例，A线程释放锁后，状态示意图如图所示

```
class MonitorExample {
    int a = 0;
    public synchronized void writer() { // 1
    	a++; // 2
    } // 3
    public synchronized void reader() { // 4
        int i = a; // 5
        ……
    } // 6
}
```

![锁获取状态示意图](\images\java并发编程的艺术02\锁获取状态示意图.jpg)

对比锁释放-获取的内存语义与volatile写-读的内存语义可以看出：锁释放与volatile写有相同的内存语义；锁获取与volatile读有相同的内存语义。所以锁有如下内存语义：

- 线程A释放一个锁，实质上是线程A向接下来将要获取这个锁的某个线程发出了（线程A对共享变量所做修改的）消息。
- 线程B获取一个锁，实质上是线程B接收了之前某个线程发出的（在释放这个锁之前对共享变量所做修改的）消息。
- 线程A释放锁，随后线程B获取这个锁，这个过程实质上是线程A通过主内存向线程B发送消息。

为了用实例理解锁的内存语义，我们以 ReentrantLock 类（可重入锁，位于java.util.concurrent.locks 包）为例，详细解读。ReentrantLock 的注释表明，该类参照 monitor 模型重新实现了一遍锁的语义，同时添加了部分扩展能力。ReentrantLock 的实例由获得而未释放锁的线程持有。该类内部包含三个 final 的内部类：Sync（后面两个的父类，实现了 AbstractQueuedSynchronizer）、FairSync（公平锁）、NonfairSync（非公平锁，ReentrantLock 默认构造函数返回的是非公平锁，如果想要获取公平锁需要使用 含参构造函数）。

> 所谓公平锁指的是哪个线程先运行，那就可以先得到锁。非公平锁是不管线程是否是先运行，都是随机获得锁的。synchronized 的实现是当线程获取锁时，首先尝试以非公平锁方式获取锁，如果获取失败，则转变为公平锁。更详细的说，当一个线程想获取锁时，先试图插队，如果占用锁的线程释放了锁，下一个线程还没来得及拿锁，那么当前线程就可以直接获得锁；如果锁正在被其它线程占用，则排队，排队的时候就不能再试图获得锁了，只能等到前面所有线程都执行完才能获得锁。

当一个线程试图获取锁时，首先尝试以非公平锁方式获取锁，调用 ReentrantLock 的 lock() 方法即是调用 NonfairSync 的 lock() 方法，代码如下：

```
 final void lock() {
            if (compareAndSetState(0, 1))
           	setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }
```

其中 compareAndSetState() 方法（JDK文档对 compareAndSetState() 方法的说明如下：如果当前状态值等于预期值，则以原子方式将同步状态设置为给定的更新值。此操作具有volatile读和写的内存语义。）即为 CAS 操作，含义为如果锁的 state 为 0 表明没有该锁没有被任何线程锁定，然后线程设置为当前线程。如果失败则说明当前锁已经被线程持有（不一定是当前线程或是其他线程），则需要进一步操作，执行 acquire() 方法。acquire() 方法直接调用了 Sync 的 nonfairTryAcquire() 方法与 AbstractQueuedSynchronizer 的 acquireQueued() 方法，前者对锁的线程进行判断，如果是当前线程则对锁的 state 进行次数增加操作，否则将该线程放入等待队列中转变为公平锁。而当 ReentrantLock 的实例是公平锁的时候，不会调用 compareAndSetState() 方法，而直接调用 acquire() 方法。

所以，加锁实现的关键在于 state 这个记录了重入次数的变量，而 state 变量以 private volatile int 修饰，所以锁的加锁得以实现。

当一个线程释放锁的时候，最终会调用 Sync 的 tryRelease(int releases) 释放锁，源码如下：

```
 protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```

上述代码的最后，写volatile变量state来实现释放锁。根据 volatile 的happens-before规则，释放锁的线程在写volatile变量之前可见的共享变量，在获取锁的线程读取同一个volatile变量后将立即变得对获取锁的线程可见，从而通知了其他线程锁已经被释放。

从对ReentrantLock的分析可以看出，锁释放-获取的内存语义的实现至少有下面两种方式。

1. 利用volatile变量的写-读所具有的内存语义。
2. 利用CAS所附带的volatile读和volatile写的内存语义。

由此，我们也可以知道Java线程之间的通信现在有了下面4种方式。

1. A线程写volatile变量，随后B线程读这个volatile变量。
2. A线程写volatile变量，随后B线程用CAS更新这个volatile变量。
3. A线程用CAS更新一个volatile变量，随后B线程用CAS更新这个volatile变量。
4. A线程用CAS更新一个volatile变量，随后B线程读这个volatile变量。 

而这些也是 concurrent 包源码通用实现模式的基础。concurrent 通用模式如下：

1. 首先，声明共享变量为volatile。
2. 然后，使用CAS的原子条件更新来实现线程之间的同步。
3. 同时，配合以volatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的通信。

#### final 的内存语义

JMM 中 final 对应的重排序规则如下：

1. 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
2. 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。
3. JMM禁止编译器把final域的写重排序到构造函数之外。
4. 在读一个对象的final域之前，一定会先读包含这个final 域的对象的引用。

这些重排序规则保证了多线程下看到，不同线程获取到的 final域的值不会改变。否则，final 成员在未赋值时的初始值有可能被意外读取。

#### happens-before 

happens-before 的根本目的，是在保证单线程与多线程程序能够正确执行的情况下，尽可能的让编译器与处理器对程序的执行进行优化。JMM可以通过happens-before关系向程序员提供跨线程的内存可见性保证（如果A线程的写操作a与B线程的读操作b之间存在happensbefore关系，尽管a操作和b操作在不同的线程中执行，但JMM向程序员保证a操作将对b操作可见）。

JSR-133 中对happens-before关系的定义如下：

1. 如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
2. 两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么这种重排序并不非法（也就是说，JMM允许这种重排序）。

其中，1 是JMM对程序员的承诺，2 是JMM对编译器和处理器重排序的约束原则。 happens-before关系本质上和as-if-serial语义是一回事。

- as-if-serial语义保证单线程内程序的执行结果不被改变，happens-before关系保证正确同步的多线程程序的执行结果不被改变。
- as-if-serial语义给编写单线程程序的程序员创造了一个幻境：单线程程序是按程序的顺序来执行的。happens-before关系给编写正确同步的多线程程序的程序员创造了一个幻境：正确同步的多线程程序是按happens-before指定的顺序来执行的。

as-if-serial语义和happens-before这么做的目的，都是为了在不改变程序执行结果的前提下，尽可能地提高程序执行的并行度。

JSR-133 中对 happens-before 的规则定义如下：

1. 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
2. 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
3. volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
4. 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。
5. start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。
6. join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。