## java并发容器
Java 5.0 提供了多种并发容器来改进同步容器的性能。

 **同步容器**是将所有对容器的访问都串行化，以实现他们的线程安全性。代价是严重降低并发行，当多个线程竞争容器的锁时，吞吐量将严重降低。

 **并发容器**是针对多个线程并发访问设计的。

Java 5.0 还增加了两中心的容器类型：Queue 和 BlockingQueue。

早期版本的JDK提供的同步容器类为Vector和Hashtable，JDK1.2 提供了Collections.synchronizedXxx等工程方法，将普通的容器继续包装。对每个共有方法都进行同步。

同步容器都是线程安全的，但是对于复合操作，有些缺点：
1. 性能差
2. 存在偷偷使用迭代器的情况，比如hashCode和equals方法也会间接地调用迭代。类似地，contailAll、removeAll、retainAll方法，以及容器作为参数的构造函数，都会对容器进行迭代。而使用迭代器，就要加锁从而很影响性能
3. 复合操作不线程安全

上述的第三点，比如说
``` java
public static Object getLast(Vector list) {
int lastIndex = list.size() - 1;
return list.get(lastIndex);
}
public static void deleteLast(Vector list) {
int lastIndex = list.size() - 1;
list.remove(lastIndex);
}
```

一般我们都是if判断是否存在某个值，再去操作，如若两个函数分别在两个线程进行。执行时，deleteLast先拿到锁，把一个元素删了。这个时候排队的get函数获得锁时则会发生拿不到对应元素从而发生错误的情况。


正是由于同步容器类有以上问题，导致这些类成了鸡肋，于是Java 5推出了并发容器类，Map对应的有**ConcurrentHashMap**，List对应的有**CopyOnWriteArrayList**。与同步容器类相比，它有以下特性：

### ConcurrentHashMap
ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了**锁分离**技术。它使用了多个锁来控制对hash表的不同部分进行的修改。ConcurrentHashMap内部使用段(Segment)来表示这些不同的部分，每个段其实就是一个小的Hashtable，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。

有些方法需要跨段，比如size()和containsValue()，它们可能需要锁定整个表而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁。这里“按顺序”是很重要的，否则极有可能出现死锁

现在稍微说下具体原理

![](image/concurrentHashmap.png)

从图中可以看到，ConcurrentHashMap内部分为很多个Segment，每一个Segment拥有一把锁，然后每个Segment（继承ReentrantLock）
