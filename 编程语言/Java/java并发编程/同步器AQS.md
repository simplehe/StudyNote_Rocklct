## java同步器AQS
在java.util.concurrent.locks包中有很多Lock的实现类，常用的有ReentrantLock、ReadWriteLock（实现类ReentrantReadWriteLock），**内部实现都依赖AbstractQueuedSynchronizer类**

队列同步器AQS是用来构建锁或其他同步组件的基础框架，内部使用一个int成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作，其中内部状态state，等待队列的头节点head和尾节点head，都是通过volatile修饰，保证了多线程之间的可见。

同步器的主要使用方式是继承，一般作为同步器组件的静态内部类，在同步器中**仅定义了与状态相关的方法**，且这个状态既可以**独占地获取又可以共享的获取**，这样就可以实现不同类型的同步组件（ReetrantLock、ReetrantReadWriteLock和CountDownLatch等）。同步器是同步组件实现锁的关键，我们通常使用同步组件来实现各种锁的功能，而其内部实际上是利用同步器进行锁的实现。它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作。

``` java
public abstract class AbstractQueuedSynchronizer extends
    AbstractOwnableSynchronizer implements java.io.Serializable {
    //等待队列的头节点
    private transient volatile Node head;
    //等待队列的尾节点
    private transient volatile Node tail;
    //同步状态
    private volatile int state;
    protected final int getState() { return state;}
    protected final void setState(int newState) { state = newState;}
    ...
}

```

由图可知维护了一个队列的头结点，尾节点，还有一个整形state。**当有线程获取锁失败后,就被添加到队列末尾**


![](image/aqs0.png)

这个队列也就是**CLH队列**!

**队列将线程封装到了Node里面**，并封装了阻塞线程和解除阻塞的操作。这个队列的本质是双向链表。Node的插入和移除都是要保证原子性的，**使用的是CAS来操作**。这个Node还封装了**线程的状态**，用字段**waitStatus**表示

zwlj：其实就是一个线程排队队列，按照一定的规则(公平锁和非公平锁)，线程可以获得锁，获得不到锁的就要阻塞并且再队列里排队。

每个节点中， 除了存储了当前线程，前后节点的引用以外，还有一个**waitStatus变量**，用于描述节点当前的状态。多线程并发执行时，队列中会有多个节点存在，这个waitStatus其实代表对应线程的状态：有的线程可能获取锁因为某些原因放弃竞争；有的线程在等待满足条件，满足之后才能执行等等。一共有4中状态：

 - CANCELLED 取消状态
 - SIGNAL 等待触发状态
 - CONDITION 等待条件状态
 - PROPAGATE 状态需要向后传播

#### 可重入锁的实现
内部会有个status变量做记录，当同一个线程再次获得锁的时候，就把statue变量累加并获得锁，方法退出时，变量减一，到达0值时则释放锁。

### 成员函数

``` java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
protected int tryAcquireShared(int arg) {
    throw new UnsupportedOperationException();

}
protected boolean tryReleaseShared(int arg) {
    throw new UnsupportedOperationException();
}
```

以上方法都是没有实现的，**AQS框架留给子类去实现**，它提供了类似aquire(),release()等方法，这些方法只是规定了基本流程，如**aquire()规定了获取独占锁的步骤**，方法里仍然依赖子类的实现

把具体的实现留给子类，父类定义基本算法流程，这是设计模式中典型的**模板模式**!

### 独占和共享
AQS定义两种资源共享方式：Exclusive（独占，只有一个线程能执行，如ReentrantLock）和Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）。

对于独占模式，AQS给出了基本的几种操作的流程：

 - acpuire() 获取锁，失败就阻塞
 - acquireInterruptibly(int arg) 获取锁，可中断
 - tryAcquireNanos(int arg, long nanosTimeout) 带有超时的阻塞模式获取锁
 - release() 释放锁

共享模式也定了几种基本操作：

 - acquireShared(int arg) 获取锁，失败就阻塞
 - acquireSharedInterruptibly(int arg) 可中断获取锁
 - tryAcquireSharedNanos(int arg, long nanosTimeout) 超时模式的获取锁
 - releaseShared(int arg) 释放共享锁

### 小结
由上可知，作为一个框架，**AQS制定了一些基本流程**，但具体的实现会不同，因此，它使用了 **模板模式(重)** 把一些细节的实现留给子类(各种Lock和并发组件)，它基本包含三个功能：

1. 同步器基本范式、结构

2. 线程的阻塞、唤醒机制

3. 线程阻塞队列的维护

### 与Synchronize的对比
JUC包中，Lock完全用Java写成，在java这个层面是无关JVM实现的。
在java.util.concurrent.locks包中有很多Lock的实现类，常用的有ReentrantLock、ReadWriteLock（实现类ReentrantReadWriteLock），**其实现都依赖AbstractQueuedSynchronizer类**，也就是这节所说的AQS框架。

AbstractQueuedSynchronizer通过**构造一个基于阻塞的CLH队列容纳所有的阻塞线程**，而对该队列的操作均通过Lock-Free（CAS）操作，但对已经获得锁的线程而言，ReentrantLock实现了偏向锁的功能。

synchronized的底层也是一个基于CAS操作的等待队列，但JVM实现的更精细，把等待队列分为ContentionList和EntryList，目的是为了降低线程的出列速度；当然也实现了偏向锁，从数据结构来说二者设计没有本质区别。但synchronized还实现了自旋锁，并针对不同的系统和硬件体系进行了优化，而Lock则完全依靠系统阻塞挂起等待线程。

**zwlj：也就是从这个层面来说，JUC包的各个并发组件，就是交由Java语言层面上的AQS框架实现的。Synchronize则是交由JVM底层实现的(利用对象头)。**
