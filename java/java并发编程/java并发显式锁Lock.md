## Lock锁
Java 5 中引入了新的锁机制——java.util.concurrent.locks 中的显式的互斥锁：Lock 接口，它提供了比synchronized 更加广泛的锁定操作。Lock 接口有 3 个实现它的类：**ReentrantLock**、**ReetrantReadWriteLock.ReadLock** 和 **ReetrantReadWriteLock.WriteLock**，即重入锁、读锁和写锁。

lock必须被显式地创建、锁定和释放，为了可以使用更多的功能，一般用ReentrantLock为其实例化。为了保证锁最终一定会被释放（可能会有异常发生），要把**互斥区放在try语句块内，并在finally语句块中释放锁**，尤其当有return语句时，return语句必须放在try字句中，以确保unlock（）不会过早发生，从而将数据暴露给第二个任务。因此，采用lock加锁和释放锁的一般形式如下：
