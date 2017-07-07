## BlockingQueue
在新增的Concurrent包中，BlockingQueue很好的解决了多线程中，如何高效安全“传输”数据的问题。通过这些高效并且线程安全的队列类，为我们快速搭建高质量的多线程程序带来极大的便利。

阻塞队列，顾名思义，首先它是一个队列，而一个队列在数据结构中所起的作用大致如下图所示：

![](image/blockingqueue1.jpg)

从上图我们可以很清楚看到，通过一个共享的队列，可以使得数据由队列的一端输入，从另外一端输出；
多线程环境中，通过队列可以很容易实现数据共享，比如**经典的“生产者”和“消费者”模型**中，通过队列可以很便利地实现两者之间的数据共享。

**阻塞队列主要用在生产者/消费者的场景**，下面这幅图展示了一个线程生产、一个线程消费的场景：

![](image/blockingqueue2.jpg)
负责生产的线程不断的制造新对象并插入到阻塞队列中，直到达到这个队列的上限值。队列达到上限值之后生产线程将会被阻塞，直到消费的线程对这个队列进行消费。同理，负责消费的线程不断的从队列中消费对象，直到这个队列为空，当队列为空时，消费线程将会被阻塞，除非队列中有新的对象被插入。

### BlockingQueue接口中的方法
在JUC包里，BlockingQueue只是一个接口。

阻塞队列一共有四套方法分别用来进行insert、remove和examine，当每套方法对应的操作不能马上执行时会有不同的反应，下面这个表格就分类列出了这些方法：

<table>
<thead>
<tr>
  <th>-</th>
  <th>Throws Exception</th>
  <th>Special Value</th>
  <th>Blocks</th>
  <th>Times Out</th>
</tr>
</thead>
<tbody><tr>
  <td>Insert</td>
  <td>add(o)</td>
  <td>offer(o)</td>
  <td>put(o)</td>
  <td>offer(o, timeout, timeunit)</td>
</tr>
<tr>
  <td>Remove</td>
  <td>remove(o)</td>
  <td>poll()</td>
  <td>take()</td>
  <td>poll(timeout, timeunit)</td>
</tr>
<tr>
  <td>Examine</td>
  <td>element()</td>
  <td>peek()</td>
  <td></td>
  <td></td>
</tr>
</tbody></table>

这四套方法对应的特点分别是：
1. ThrowsException：如果操作不能马上进行，则抛出异常
2. SpecialValue：如果操作不能马上进行，将会返回一个特殊的值，一般是true或者false
3. Blocks:如果操作不能马上进行，操作会被阻塞
4. TimesOut:如果操作不能马上进行，操作会被阻塞指定的时间，如果指定时间没执行，则返回一个特殊值，一般是true或者false

需要注意的是，我们不能向BlockingQueue中插入null，否则会报NullPointerException。

### 实现类
BlockingQueue只是java.util.concurrent包中的一个接口，而在具体使用时，我们用到的是它的实现类，当然这些实现类也位于java.util.concurrent包中。在Java6中，BlockingQueue的实现类主要有以下几种：
1. ArrayBlockingQueue
2. DelayQueue
3. LinkedBlockingQueue
4. PriorityBlockingQueue
5. SynchronousQueue

#### ArrayBlockingQueue
ArrayBlockingQueue是一个**有边界的阻塞队列**，它的内部实现是一个数组。有边界的意思是它的容量是有限的，我们必须在其初始化的时候**指定它的容量大小**，容量大小一旦指定就不可改变。

下面是一个例子
``` java
BlockingQueue queue = new ArrayBlockingQueue(1024);
queue.put("1");
Object object = queue.take();
```

#### DelayQueue
DelayQueue阻塞的是其内部元素，DelayQueue中的元素必须实现 java.util.concurrent.Delayed接口，这个接口的定义非常简单：
``` java
public interface Delayed extends Comparable<Delayed> {
long getDelay(TimeUnit unit);
}
```

getDelay()方法的返回值就是队列元素被释放前的保持时间，如果返回0或者一个负值，就意味着该元素已经到期需要被释放，此时DelayedQueue会通过其take()方法释放此对象。

zwlj：也就是说，某个对象(实现delay的对象)进入某个delay队列时，相当于立刻被锁住，要经过一定的时间才能解锁，然后出队。

从上面Delayed 接口定义可以看到，它还继承了Comparable接口，这是因为DelayedQueue中的元素需要进行排序，一般情况，我们都是按元素过期时间的优先级进行排序。

接下来实例定义一个delay元素
``` java
public class DelayedElement implements Delayed {
  private long expired;
  private long delay;
  private String name;

  DelayedElement(String elementName, long delay) {
         this. name = elementName;
         this. delay= delay;
         expired = ( delay + System. currentTimeMillis());
  }

  @Override
  public int compareTo(Delayed o) {
        DelayedElement cached=(DelayedElement) o;
         return cached.getExpired()> expired?1:-1;
  }

  @Override
  public long getDelay(TimeUnit unit) {

         return ( expired - System. currentTimeMillis());
  }

  @Override
  public String toString() {
         return "DelayedElement [delay=" + delay + ", name=" + name + "]";
  }

  public long getExpired() {
         return expired;
  }

}
```

可以看到它实现了getDelay方法，初始化时获取当前时间加上delay值为过期时间。然后compare可以用这个过期时间来排序。

``` java
public class DelayQueueExample {
  public static void main(String[] args) throws InterruptedException {
        DelayQueue<DelayedElement> queue= new DelayQueue<>();
        DelayedElement ele= new DelayedElement( "cache 3 seconds",3000);
         queue.put( ele);
        System. out.println( queue.take());

  }
```

运行这个main函数，我们可以发现，我们需要等待3s之后才会打印这个对象。

其实DelayQueue应用场景很多，比如定时关闭连接、缓存对象，超时处理等各种场景.

#### LinkedBlockingQueue
LinkedBlockingQueue阻塞队列大小的配置是可选的，如果我们初始化时指定一个大小，它就是有边界的，如果不指定，它就是无边界的。说是无边界，其实是采用了默认大小为Integer.MAX_VALUE的容量 。它的内部实现是一个链表。

和ArrayBlockingQueue一样，LinkedBlockingQueue 也是以先进先出的方式存储数据，最新插入的对象是尾部，最新移出的对象是头部。

####  PriorityBlockingQueue
PriorityBlockingQueue是一个没有边界的队列，它的排序规则和 java.util.PriorityQueue一样。需要注意，PriorityBlockingQueue中允许插入null对象。

所有插入PriorityBlockingQueue的对象必须实现 java.lang.Comparable接口，队列优先级的排序规则就是按照我们对这个接口的实现来定义的。

另外，我们可以从PriorityBlockingQueue获得一个迭代器Iterator，但这个迭代器并不保证按照优先级顺序进行迭代。

#### SynchronousQueue
SynchronousQueue队列内部仅允许容纳一个元素。当一个线程插入一个元素后会被阻塞，除非这个元素被另一个线程消费。

意思就是每个 put 必须等待一个 take，反之亦然。同步队列没有任何内部容量，甚至连一个队列的容量都没有。当一个任务在任务队列里的时候，如果生产者不把这个任务带走,那么这个任务将会一直卡在这个队列里，别的任务也进不了。
