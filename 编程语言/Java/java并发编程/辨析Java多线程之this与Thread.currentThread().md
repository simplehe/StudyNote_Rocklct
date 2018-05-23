## 辨析Java多线程之this与Thread.currentThread()

**Thread.currentThread()是Thread类中的一个静态方法，用来获得当前线程这个对象**,也就是获得一个Thread类。这总是容易和this搞混，因为往往会出现一种情况currentThread拿到的Thread对象和this获得的对象不对应。

**注，currentThread方法是native方法，实现由c完成，所以如何获取当前线程实现细节暂不考虑**

下面耐心读完下面2段代码：
``` java
package mythread;

public class CountOperate extends Thread{

    public CountOperate(){
        System.out.println("CountOperate---begin");
        System.out.println("Thread.currentThread().getName()=" + Thread.currentThread().getName());//获取线程名
        System.out.println("Thread.currentThread().isAlive()=" + Thread.currentThread().isAlive()); //查看线程是否存活
        System.out.println("this.getName=" + this.getName());
        System.out.println("this.isAlive()=" + this.isAlive());
        System.out.println("CountOperate---end ");
        System.out.println("Thread.currentThread()==this :"+ (Thread.currentThread() == this));
    }

    @Override
    public void run() {
        System.out.println("run---begin");
        System.out.println("Thread.currentThread().getName=" + Thread.currentThread().getName());
        System.out.println("Thread.currentThread().isAlive()" + Thread.currentThread().isAlive());
        System.out.println("Thread.currentThread()==this :"+ (Thread.currentThread() == this));
        System.out.println("this.getName()=" + this.getName());
        System.out.println("this.isAlive()=" + this.isAlive());
        System.out.println("run --- end");
    }
}
```

主线程：
``` java
public class Run {

    public static void main(String[] args){

        CountOperate c = new CountOperate();
        Thread t1 = new Thread(c);
        System.out.println("main begin t1 isAlive=" + t1.isAlive());
        t1.setName("A");
        t1.start();
        System.out.println("main end t1 isAlive=" + t1.isAlive());

    }
}
```

上面代码的输出(不含注释)
```
//初始化
CountOperate---begin
Thread.currentThread().getName()=main
Thread.currentThread().isAlive()=true
this.getName=Thread-0
this.isAlive()=false
CountOperate---end
Thread.currentThread()==this :false

//初始化完继续跑主线程
main begin t1 isAlive=false
main end t1 isAlive=true
run---begin
Thread.currentThread().getName=A
Thread.currentThread().isAlive()true
Thread.currentThread()==this :false
this.getName()=Thread-0
this.isAlive()=false
run --- end


```

#### 代码阅读-线程初始化

看到代码不要急躁，来耐心分析这段代码。先从主线程Run类开始看。

首先是初始化了一个CountOperate对象，这个对象初始化时干了四件事，用currentThread和this分别获取对象，然后判断存不存活，并且两者相不相等。

结果表明，此时线程获取到的是主线程，而this获得的是当前方法所在实例也就是Thread-0(**Thread在生成时都会有方法默认赋予其名字**)，两者不想等。这个很好理解，因为构造出这个Thread对象(c对象)的线程是主线程，currentThread当然可以获得主线程，而这里的this是这个构造方法调用时的这个实例。那么当然获取到的是**c**这个对象，也就是Thread-0

此时c这个对象并没有start，所以这个时候线程当然是不存活的。

#### 代码阅读-主线程后半部分

接下来继续跑主线程这个部分，这里新建了一个线程t1，并传入c线程对象初始化。那么这个的实际含义是什么呢，我们可以看源码

![](image/currenthread0.png)

这个初始化，传入的参数c就是代码里的target。意思便是，用线程初始化时，会有一个private变量target，并把这个用于初始化的线程赋给target。也就是**this.target = c;**

**那么在线程start的时候就是直接执行的target.run(),直接拿target方法中的run方法来用，而不是新建一个target的线程，不是target.start();**

回到上面代码的执行结果上面来，程序先是给t1赋了个名字，然后让t1跑起来，运行run方法。

此时输出结果是currentThread获取的线程名字是A，存活状态。而this获得的确实Thread-0，也就是对象c，状态为不存活。

这里就必须辨析清楚了，因为这个时候是t1这个线程在运行，并且他执行的是target.run()方法，也就是说实际上是t1拿到c的实例在跑c的方法。所以currentread拿到的当然就是被setName之后的A，而run方法是在c里的，这个时候this获得的实例当然还是c，也就是target。所以this的名字是thread0，而且此时c并没有start，所以c状态不存活。

#### 新例

Run类代码进行一下修改，也就是稍微修改下主线程
```
public class Run {

    public static void main(String[] args){

        CountOperate c = new CountOperate();
        c.start();
    }
}
```

打印的结果为：
```
CountOperate---begin
Thread.currentThread().getName()=main
Thread.currentThread().isAlive()=true
this.getName=Thread-0
this.isAlive()=false
CountOperate---end
Thread.currentThread()==this :false

run---begin
Thread.currentThread().getName=Thread-0
Thread.currentThread().isAlive()true
Thread.currentThread()==this :true
this.getName()=Thread-0
this.isAlive()=true
run --- end
```

c在start之后，就新生成了一个线程c，这个时候无论是this还是currentthread都指向了一个对象，所以两者相等，并且状态存活。
