## Condition
Condition是在Java 1.5中才出现的，它用来替代传统的Object的wait()、notify()实现线程间的协作，相比使用Object的wait()、notify()，使用Condition的await()、signal()这种方式实现线程间协作更加安全和高效。因此通常来说比较推荐使用Condition，阻塞队列实际上是使用了Condition来模拟线程间协作。

 - Condition是个接口，基本的方法就是await()和signal()方法；
 - Condition依赖于Lock接口，生成一个Condition的基本代码是lock.newCondition()
 - 调用Condition的await()和signal()方法，都必须在lock保护之内，就是说必须在lock.lock()和lock.unlock之间才可以使用

 Conditon中的await()对应Object的wait()；
 Condition中的signal()对应Object的notify()；

 接下来看一个典型的生产者消费者的demo
 ``` java
 class BoundedBuffer {
  final Lock lock = new ReentrantLock();//锁对象
  final Condition notFull = lock.newCondition();//写线程条件
  final Condition notEmpty = lock.newCondition();//读线程条件

  final Object[] items = new Object[100];//缓存队列
  int putptr/*写索引*/, takeptr/*读索引*/, count/*队列中存在的数据个数*/;

  public void put(Object x) throws InterruptedException {
   lock.lock();
   try {
    while (count == items.length)//如果队列满了
     notFull.await();//阻塞写线程
    items[putptr] = x;//赋值
    if (++putptr == items.length) putptr = 0;//如果写索引写到队列的最后一个位置了，那么置为0
    ++count;//个数++
    notEmpty.signal();//唤醒读线程
   } finally {
    lock.unlock();
   }
  }

  public Object take() throws InterruptedException {
   lock.lock();
   try {
    while (count == 0)//如果队列为空
     notEmpty.await();//阻塞读线程
    Object x = items[takeptr];//取值
    if (++takeptr == items.length) takeptr = 0;//如果读索引读到队列的最后一个位置了，那么置为0
    --count;//个数--
    notFull.signal();//唤醒写线程
    return x;
   } finally {
    lock.unlock();
   }
  }
 }

 ```

 可以看到上面代码，notFull和notEmpty两个condition绑定的都是一个lock对象。这就是lock对象相比于Sychronize关键字的一个巨大优势，一个锁可以绑定多个条件。

 **自我总结**，对于上面代码，具体来说，两个condition的意义在哪里呢？难道用Synchronize就不可以实现生产者消费者吗？并非如此，如果用Synchronize锁住put和get方法，一样可以实现生产者消费者，但是这又一个缺点，假想一下有很多个线程进行生产消费，加入用Synchronize，生产者生产以后会会用notify唤醒一个线程，此时这个线程不保证是消费者，可能还是生产者，假如此时buffer已经满了的话，不停的唤醒生产者而又不能生产继续进入等待队列，这是一种资源浪费。**而如果使用两个Condition**，可以保证生产完之后唤醒的就是消费者，如上面代码，唤醒的是对应condition的等待队列，这样就优化了性能。
