## Executor框架

首先我们在之前的笔记JUC包基础概述里已经知道了，JUC包的大致结构。从功能上大概可以分为那几个模块，那么这里就要提到**Executor框架**。这个概念了。

什么是Executor框架呢？这个框架其实就是指JUC包里的：线程池，Executor(JUC包下的一个接口)，Executors(JUC包下的一个类)，ExecutorService，CompletionService，Future，Callable等。

以上这些JUC包里的东西可以被统称为Executor框架。

首先来观察一下Executor这个接口继承关系的整体架构

![](image/Executor.png)

Executor 接口中之定义了一个方法 execute（Runnable command），该方法接收一个 Runable 实例，它用来执行一个任务，任务即一个实现了 Runnable 接口的类。

ExecutorService接口继承自Executor接口，定义了终止、提交,执行任务、跟踪任务返回结果等方法. **这个接口包含着管理线程的一些方法的定义，意味着这个接口将被各种线程池实现**

这个接口有以下方法

1. execute（Runnable command）：履行Ruannable类型的任务,
2. submit（task）：可用来提交Callable或Runnable任务，并返回代表此任务的Future对象
3. shutdown（）：在完成已提交的任务后封闭办事，不再接管新任务,
4. shutdownNow（）：停止所有正在履行的任务并封闭办事。
5. isTerminated（）：测试是否所有任务都履行完毕了。,
6. isShutdown（）：测试是否该ExecutorService已被关闭

也就是各种具体的线程池，通常都直接或间接实现了这个接口下定义的方法。

接下来讲讲Executors这个类。这个类也在JUC包下。

Executors提供了一系列工厂方法用于创先线程池，返回的线程池都实现了ExecutorService接口。也就是说Executors是一个工具类，用来创建线程池。

那么Executors便能用作创建线程池了，关于线程池的内容，可以查阅ThreadPool相关笔记。

### Future和Callback
自从Java 1.5开始，就提供了Callable和Future，通过它们可以在任务执行完毕之后得到任务执行结果。这两个类在JUC包下，实质上也属于Executor体系里的一部分。

先说一下java.lang.Runnable吧，它是一个接口，在它里面只声明了一个run()方法.由于run()方法返回值为void类型，所以在执行完任务之后无法返回任何结果。

#### Callable
Callable位于java.util.concurrent包下，它也是一个**接口**，在它里面也只声明了一个方法，只不过这个方法叫做call()：

那么怎么使用Callable呢？一般情况下是配合ExecutorService来使用的，在ExecutorService接口中声明了若干个submit方法的重载版本：


``` java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
```

#### Future
Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。

Future位于java.util.concurrent包下，它是一个接口：
``` java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

总的来说便是以上几个方法

 - cancel方法用来取消任务，如果取消任务成功则返回true，如果取消任务 失败则返回false。参数mayInterruptIfRunning表示是否允许取消正在执行却没有执行完毕的任务，如果设置true，则表示可以取消正在执行过程中的任务。如果任务已经完成，则无论mayInterruptIfRunning为true还是false，此方法肯定返回false，即如果取消已经完成的任务会返回false；如果任务正在执行，若mayInterruptIfRunning设置为true，则返回true，若mayInterruptIfRunning设置为false，则返回false；如果任务还没有执行，则无论mayInterruptIfRunning为true还是false，肯定返回true。
 - isCancelled方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。
 - isDone方法表示任务是否已经完成，若任务完成，则返回true；
 - get()方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；
 - get(long timeout, TimeUnit unit)用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。


 也就是说Future提供了三种功能：

 1. 判断任务是否完成；

 2. 能够中断任务；

 3. 能够获取任务执行结果。

 因为Future只是一个接口，所以是无法直接用来创建对象使用的，因此就有了下面的FutureTask。

 #### FutrureTask
 FutureTask是一个实现类，它的实现：
 ``` java
 public class FutureTask<V> implements RunnableFuture<V>
 ```

FutureTask类实现了RunnableFuture接口，我们看一下RunnableFuture接口的实现：
``` java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```
事实上，FutureTask是Future接口的一个唯一实现类。

那么可以通过例子来阐述一下基本用法

使用Callable和Future获取执行结果：
``` java
public class Test {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        Task task = new Task();
        Future<Integer> result = executor.submit(task);
        executor.shutdown();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }

        System.out.println("主线程在执行任务");

        try {
            System.out.println("task运行结果"+result.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

        System.out.println("所有任务执行完毕");
    }
}
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        System.out.println("子线程在进行计算");
        Thread.sleep(3000);
        int sum = 0;
        for(int i=0;i<100;i++)
            sum += i;
        return sum;
    }
}
```

以上首先创建了一个实现Callable接口的对象Task,然后提交到线程池里获得一个future，然后从这个future里get出结果。get的时候是阻塞至得到结果为止。
