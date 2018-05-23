## Java 字符串（String, StringBuffer, StringBuilder）

说到java字符串，我们常遇到3个。

 - String 字符串常量
 - StringBuffer 字符串变量（线程安全）
 - StringBuilder 字符串变量（非线程安全）

![](image/string0.png)


查看 API 会发现，String、StringBuffer、StringBuilder 都实现了**CharSequence 接口**，内部都是用一个char数组实现，虽然它们都与字符串相关，但是其处理机制不同。

 - String：是不可改变的量，也就是创建后就不能在修改了。

 - StringBuffer：是一个可变字符串序列，它与 String 一样，在内存中保存的都是一个有序的字符串序列（char 类型的数组），不同点是 StringBuffer 对象的值都是可变的。

 - StringBuilder：与 StringBuffer 类基本相同，都是可变字符换字符串序列，不同点是 **StringBuffer 是线程安全的，StringBuilder 是线程不安全的**。

### StringBuffer 使用

``` java
public StringBuffer append(type t);//追加
public StringBuffer insert(int offset,type t);//插入，在offset处加入类型为type的字符串

public StringBuffer reverse(); //反转
public StringBuffer delete(int start,int end); //删除

char charAt(int index)
//返回此序列中指定索引处的 char 值。
void setCharAt(int index, char ch)
//设置此序列中指定索引处的 char 值。
```

### 使用场景
使用 String 类的场景：在字符串**不经常变化**的场景中可以使用 String 类，例如常量的声明、少量的变量运算。

使用 StringBuffer 类的场景：在**频繁进行字符串运算**（如拼接、替换、删除等），并且运行在**多线程环境**中，则可以考虑使用 StringBuffer，例如 **XML 解析、HTTP 参数解析和封装**。

使用 StringBuilder 类的场景：在频繁进行字符串运算（如拼接、替换、和删除等），并且**运行在单线程的环境**中，则可以考虑使用 StringBuilder，如**SQL 语句的拼装、JSON 封装等**。

### 分析
在性能方面，由于 String 类的操作是产生新的 String 对象，而 StringBuilder 和 StringBuffer 只是一个字符数组的扩容而已，所以 **String 类的操作要远慢于 StringBuffer 和 StringBuilder**。

简要的说， String 类型和 StringBuffer 类型的主要性能区别其实在于 **String 是不可变的对象**, 因此在每次对 String 类型进行改变的时候其实都等同于**生成了一个新的 String 对象，然后将指针指向新的 String 对象**。所以经常改变内容的字符串最好不要用 String ，因为每次生成对象都会对系统性能产生影响，特别当内存中无引用对象多了以后， JVM 的 GC 就会开始工作，那速度是一定会相当慢的。


但是，**在某些特别情况下**， String 对象的字符串拼接其实是被 JVM 解释成了 StringBuffer 对象的拼接，所以这些时候 String 对象的速度并不会比 StringBuffer 对象慢，而特别是以下的字符串对象生成中， String 效率是远要比 StringBuffer 快的

``` java
//1
String S1 = "This is only a " +  "simple" + "  test";

//2
StringBuffer Sb = new StringBuilder("This is only a").append(" simple ").append("test");

```

会很惊讶的发现，生成 String S1 对象的速度简直太快了，而这个时候 StringBuffer 居然速度上根本一点都不占优势。其实这是 JVM 的一个把戏，在 JVM 眼里，这个

String S1 = “This is only a" + “ simple" + “test";

其实就是

String S1 = “This is only a simple test";

所以当然不需要太多的时间了。但大家这里要注意的是，如果你的字符串是来自另外的 String 对象的话，速度就没那么快了

### 总结

**在大部分情况下 StringBuffer > String**

Java.lang.StringBuffer 是线程安全的可变字符序列。一个类似于 String 的字符串缓冲区，但不能修改。虽然在任意时间点上它都包含某种特定的字符序列，但通过某些方法调用可以改变该序列的长度和内容。在程序中可将字符串缓冲区安全地用于多线程。而且在必要时可以对这些方法进行同步，因此任意特定实例上的所有操作就好像是以串行顺序发生的，该顺序与所涉及的每个线程进行的方法调用顺序一致。

**在大部分情况下 StringBuilder > StringBuffer**

java.lang.StringBuilder 一个可变的字符序列是 JAVA 5.0 新增的。此类提供一个与 StringBuffer 兼容的 API，但不保证同步，所以使用场景是单线程。该类被设计用作 StringBuffer 的一个简易替换，用在字符串缓冲区被单个线程使用的时候（这种情况很普遍）。**如果可能，建议优先采用该类，因为在大多数实现中，它比 StringBuffer 要快。两者的使用方法基本相同**。
