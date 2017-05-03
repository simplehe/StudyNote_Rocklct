## Object类
Object类是所有Java类的祖先。每个类都使用 Object 作为超类。所有对象（包括数组）都实现这个类的方法。在不明确给出超类的情况下，Java会自动把Object作为要定义类的超类。

它提供了**11**个方法

1. public final native Class<?> getClass()
2. public native int hashCode()
3. public boolean equals(Object obj)
4. protected native Object clone() throws CloneNotSupportedException
5. public String toString()
6. public final native void notify()
7. public final native void notifyAll()
8. public final native void wait(long timeout) throws InterruptedException
9. public final void wait(long timeout, int nanos) throws InterruptedException
10. public final void wait() throws InterruptedException
11. protected void finalize() throws Throwable { }

接下来逐个查看

### getClass方法
getClass方法是一个final方法，不允许子类重写，并且也是一个native方法。

返回当前运行时对象的Class对象，注意这里是运行时

比如以下代码中n是一个Number类型的实例，但是java中数值默认是Integer类型，所以getClass方法返回的是java.lang.Integer：

``` java
"str".getClass() // class java.lang.String
"str".getClass == String.class // true
Number n = 0;
Class<? extends Number> c = n.getClass(); // class java.lang.Integer
```

### hasCode方法
hashCode方法也是一个native方法。

该方法返回对象的哈希码，主要使用在哈希表中，比如JDK中的HashMap。

哈希码有通用约定

- 在java程序执行过程中，在一个对象没有被改变的前提下，无论这个对象被调用多少次，hashCode方法都会返回相同的**整数值**。对象的哈希码没有必要在不同的程序中保持相同的值。
- 如果2个对象使用equals方法进行比较并且相同的话，那么这2个对象的hashCode方法的值也**必须相等**。
- 如果根据equals方法，得到两个对象不相等，那么这2个对象的hashCode值**不需要必须不相同**。但是，不相等的对象的hashCode值不同的话可以提高哈希表的性能。

example,以下是String类的hashCode方法
``` java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value; //获取String类里的char数组

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

原理是 h = s[0]\*31^(n-1) + s[1]\*31^(n-2) + … + s[n-1]\*31^0

有点像ascill码配合31进制。

自定义对象需要重写equals方法和hashcode方法，不过也要注意，hashcode的算法最好必须满足上面列出来的三条通用约定。如果两个对象equals一样，但是hashcode不同，会导致hashmap里冗余存储，如果两个对象equals不一样，hashcode一样，这无所谓，只不过会导致性能降低而已。

建议2个方法equals和hashCode都一起重写

### equals方法
比较两个对象是否相等。Object类的默认实现，即比较2个对象的内存地址是否相等：
``` java
public boolean equals(Object obj) {
    return (this == obj);
}
```

对于任何非空引用值 x，x.equals(null) 都应返回 false

注意点：如果重写了equals方法，通常有必要重写hashCode方法，这点已经在hashCode方法中说明了。

### clone方法
创建并返回当前对象的一份拷贝。一般情况下，对于任何对象 x，表达式 x.clone() != x 为true，x.clone().getClass() == x.getClass() 也为true。

Object类的clone方法是一个protected的native方法。

由于Object本身没有实现Cloneable接口，所以不重写clone方法并且进行调用的话会发生CloneNotSupportedException异常。

### toString方法
Object对象的默认实现，即输出类的名字@实例的哈希码的16进制：
``` java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

toString方法的结果应该是一个简明但易于读懂的字符串。建议Object所有的子类都重写这个方法。

### notify方法

notify方法是一个native方法，并且也是final的，不允许子类重写。

唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果所有的线程都在此对象上等待，那么只会选择一个线程。选择是任意性的，并在对实现做出决定时发生。一个线程在对象监视器上等待可以调用wait方法。

直到当前线程放弃对象上的锁之后，被唤醒的线程才可以继续处理。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争。例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。

notify方法只能被作为此对象监视器的所有者的线程来调用。一个线程要想成为对象监视器的所有者，可以使用以下3种方法：

1. 执行对象的同步实例方法
2. 使用synchronized内置锁
3. 对于Class类型的对象，执行同步静态方法

一次只能有一个线程拥有对象的监视器。

如果当前线程不是此对象监视器的所有者的话会抛出IllegalMonitorStateException异常