## RandomAccessFile
IO包下的一个类。

RandomAccessFile 是随机访问文件(包括读/写)的类。它支持对文件随机访问的读取和写入，即我们可以从指定的位置读取/写入文件数据。

它是一个完全独立的类，所有方法(绝大多数都只属于它自己)都是从零开始写的。

需要注意的是，RandomAccessFile 虽然属于java.io包，但它不是InputStream或者OutputStream的子类；它也不同于FileInputStream和FileOutputStream。 FileInputStream 只能对文件进行读操作，而FileOutputStream 只能对文件进行写操作；但是，**RandomAccessFile 同时支持文件的读和写，并且它支持随机访问**。

意思就是这样一个文件类，他可以支持在文件的特定位置里，进行读写操作。

文件内容本身，就类似于一个大的字节数组。

好比我有一个文件里面保存着三条记录(三行固定长度的文本数据)
![](image/RandomAccessFile.png)

这个类便可以像数组一样，如若知道索引，当然可以进行特定位置的读写操作。比如在第二行加一行啊，最后一行后面加一行。**用这个类可以避免将文件所有内容都读出来操作完后再写会文件内。**

创建文件 r 为只读；w 为只写；rw 为读写。
``` java
RandomAccessFile raf = new RandomAccessFile(file,"rw");
```


核心操作是
 - getFilePointer()	返回文件记录指针的当前位置
 - seek(long pos)	将文件记录指针定位到pos的位置

下面一个基础示例
``` java
RandomAccessFile raf=new RandomAccessFile(path, "r");
//获取RandomAccessFile对象文件指针的位置，初始位置是0
System.out.println("RandomAccessFile文件指针的初始位置:"+raf.getFilePointer());
raf.seek(20);//移动文件指针位置
byte[]  buff=new byte[1024];
//用于保存实际读取的字节数
int hasRead=0;
//循环读取
while((hasRead=raf.read(buff))>0){
//打印读取的内容,并将字节转为字符串输入
System.out.println(new String(buff,0,hasRead));
}
```

一开始获得的文件指针位置为0.然后seek用来偏移位置，从文件的第20个字节位置开始，读取数据。

虽然这个类有一定的用途，但是：

**在JDK 1.4中，RandomAccessFile的绝大多数功能（但不是全部）已经被nio内存映射文件给取代了。你该考虑一下是不是用"内存映射文件"来代替RandomAccessFile了**
