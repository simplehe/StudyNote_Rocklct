## Spring事务传播行为
Spring定义了7个传播行为

 - PROPAGATION_REQUIRED--支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。
 - PROPAGATION_SUPPORTS--支持当前事务，如果当前没有事务，就以非事务方式执行。
 - PROPAGATION_MANDATORY--支持当前事务，如果当前没有事务，就抛出异常。
 - PROPAGATION_REQUIRES_NEW--新建事务，如果当前存在事务，把当前事务挂起。
 - PROPAGATION_NOT_SUPPORTED--以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
 - PROPAGATION_NEVER--以非事务方式执行，如果当前存在事务，则抛出异常
 - PROPAGATION_NESTED--外层事务的回滚可以引起内层事务的回滚。而内层事务的异常并不会导致外层事务的回滚

这些传播行为，其实是定义了，当事务出现调用和嵌套调用的时候，应该如何处理。

下面来简单说说七种行为，**以第一种传播方式重点举例**。

### PROPAGATION_REQUIRED
如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务。

什么意思呢，假如我们有以下两个方法，这两个方法都被预先定义为事务。那么spring会帮我们怎么处理呢？

```
//事务属性 PROPAGATION_REQUIRED
methodA{
    ……
    methodB();
    ……
}

//事务属性 PROPAGATION_REQUIRED
methodB{
   ……
}
```

methodA里调用了methodB

那么如果我单独调用B方法

``` java
main(){
    metodB();
}
```
那就相当于

``` java
Main{
    Connection con=null;
    try{
        con = getConnection();
        con.setAutoCommit(false);

        //方法调用
        methodB();

        //提交事务
        con.commit();
    } Catch(RuntimeException ex) {
        //回滚事务
        con.rollback();
    } finally {
        //释放资源
        closeCon();
    }
}
```

methodb在调用时给自己封装了一个事务，这是单独调用B，而B传播机制是REQURIED的时候

如果调用的是A呢，这种情况下我们称之为，**方法A传播到方法B**。显然A是一个事务，封装成一个事务，那**我们B就选择支持(加入)这个事务就可以了**，并**不需要在里面多开一层事务把b包起来**。

``` java
main{
    Connection con = null;
    try{
        con = getConnection();
        methodA();
        con.commit();
    } catch(RuntimeException ex) {
        con.rollback();
    } finally {
        closeCon();
    }
}
```

**PROPAGATION_REQUIRED应该是我们首先的事务传播行为。它能够满足我们大多数的事务需求。**

### PROPAGATION_SUPPORTS
如果存在一个事务，支持当前事务。如果没有事务，则非事务的执行。但是对于事务同步的事务管理器，PROPAGATION_SUPPORTS与不使用事务有少许不同。

在理解了PROPAGATION_REQUIRED的例子情况下。

```
//事务属性 PROPAGATION_REQUIRED
methodA{
    ……
    methodB();
    ……
}

//事务属性 PROPAGATION_SUPPORTS
methodB{
   ……
}
```

假如直接调用B，则不进行任何封装，直接调用。假如在A里调用B，但是A属于REQUIRED，所以必然会开启一个事务包装，那么B此时就假如A。

对于SUPPORT，比较好的理解就是，有就有(支持)，没有就没有，什么都不做。

### PROPAGATION_MANDATORY
支持当前事务，如果当前没有事务，就抛出异常。

```
//事务属性 PROPAGATION_REQUIRED
methodA{
    ……
    methodB();
    ……
}

//事务属性 PROPAGATION_MANDATORY
methodB{
   ……
}
```

当单独调用methodB时，因为当前没有一个活动的事务，则会抛出异常throw new IllegalTransactionStateException(“Transaction propagation ‘mandatory’ but no existing transaction found”);当调用methodA时，methodB则加入到methodA的事务中，事务地执行。

也就是，**没有封装我为事务，就报错**，非常强硬。

### PROPAGATION_REQUIRES_NEW
总是开启一个新的事务。如果一个事务已经存在，则将这个存在的事务挂起。

``` java
//事务属性 PROPAGATION_REQUIRED
methodA(){
    doSomeThingA();
    methodB();
    doSomeThingB();
}

//事务属性 PROPAGATION_REQUIRES_NEW
methodB(){
    ……
}
```

一旦调用methodA。

``` java
main(){
    TransactionManager tm = null;
    try{
        //获得一个JTA事务管理器
        tm = getTransactionManager();
        tm.begin();//开启一个新的事务
        Transaction ts1 = tm.getTransaction();
        doSomeThing();
        tm.suspend();//挂起当前事务
        try{
            tm.begin();//重新开启第二个事务
            Transaction ts2 = tm.getTransaction();
            methodB();
            ts2.commit();//提交第二个事务
        } Catch(RunTimeException ex) {
            ts2.rollback();//回滚第二个事务
        } finally {
            //释放资源
        }
        //methodB执行完后，恢复第一个事务
        tm.resume(ts1);
        doSomeThingB();
        ts1.commit();//提交第一个事务
    } catch(RunTimeException ex) {
        ts1.rollback();//回滚第一个事务
    } finally {
        //释放资源
    }
}
```

发生这种情况会先挂起当前事务，然后开启新的事务封装B，然后执行完B以后再恢复A的事务，然后执行。

根据上面代码，在这里，我把ts1称为外层事务，ts2称为内层事务。从上面的代码可以看出，ts2与ts1是两个独立的事务，互不相干。Ts2是否成功并不依赖于 ts1。如果methodA方法在调用methodB方法后的doSomeThingB方法失败了，**而methodB方法所做的结果依然被提交**。而**除了 methodB之外的其它代码导致的结果却被回滚了**。使用PROPAGATION_REQUIRES_NEW,需要使用 JtaTransactionManager作为事务管理器。

原则就是无论怎样都要创建一个新事务，这和嵌套他的事务已经无任何关系了。

### PROPAGATION_NOT_SUPPORTED
总是非事务地执行，并挂起任何存在的事务。使用PROPAGATION_NOT_SUPPORTED,也需要使用JtaTransactionManager作为事务管理器。

（代码示例同上，可同理推出）

总之就是，没有就没有，有也不支持，把他挂起来不管他。

### PROPAGATION_NEVER
总是非事务地执行，如果存在一个活动事务，则抛出异常。

一点都不支持事务，没有就没有，有了就报错。

### PROPAGATION_NESTED
外层事务的回滚可以引起内层事务的回滚。而内层事务的异常并不会导致外层事务的回滚，**它是一个真正的嵌套事务**。

如果没有就新建事务，如果有，就开启一个新事务在里面，但是这个事务属于子事务，子事务出错不会引起主事务回滚，主事务出错会引起子事务回滚。

``` java
//事务属性 PROPAGATION_REQUIRED
methodA(){
    doSomeThingA();
    methodB();
    doSomeThingB();
}

//事务属性 PROPAGATION_NESTED
methodB(){
    ……
}
```

``` java
main(){
    Connection con = null;
    Savepoint savepoint = null;
    try{
        con = getConnection();
        con.setAutoCommit(false);
        doSomeThingA();
        savepoint = con2.setSavepoint();
        try{
            methodB();
        } catch(RuntimeException ex) {
            con.rollback(savepoint);
        } finally {
            //释放资源
        }
        doSomeThingB();
        con.commit();
    } catch(RuntimeException ex) {
        con.rollback();
    } finally {
        //释放资源
    }
}
```

当methodB方法调用之前，调用setSavepoint方法，保存当前的状态到savepoint。如果methodB方法调用失败，则恢复到之前保存的状态。但是需要注意的是，这时的事务并没有进行提交，如果后续的代码(doSomeThingB()方法)调用失败，则回滚包括methodB方法的所有操作。

嵌套事务一个非常重要的概念就是内层事务依赖于外层事务。外层事务失败时，会回滚内层事务所做的动作。而内层事务操作失败并不会引起外层事务的回滚。

DataSourceTransactionManager使用savepoint支持PROPAGATION_NESTED时，需要JDBC 3.0以上驱动及1.4以上的JDK版本支持。其它的JTA TrasactionManager实现可能有不同的支持方式。
