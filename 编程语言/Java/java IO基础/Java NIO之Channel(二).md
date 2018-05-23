## Channel
Channel是对原IO中流的模拟，任何来源和目的数据都必须通过一个Channel对象。

Channel是一个对象，可以通过它读取和写入数据。可以把它看做IO中的流。但是它和流相比还有一些不同：

 - Channel是双向的，既可以读又可以写，而流是单向的
 - Channel可以进行异步的读写
 - 对Channel的读写必须通过buffer对象

所有数据都通过Buffer对象处理，所以，您永远不会将字节直接写入到Channel中，相反，您是将数据写入到Buffer中；同样，您也不会从Channel中读取字节，而是将数据从Channel读入Buffer，再从Buffer获取这个字节。

在Java NIO中Channel主要有如下几种类型：

 - FileChannel：从文件读取数据的
 - DatagramChannel：读写UDP网络协议数据
 - SocketChannel：读写TCP网络协议数据
 - ServerSocketChannel：可以监听TCP连接


### 例子

``` java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
  FileChannel inChannel = aFile.getChannel();

  ByteBuffer buf = ByteBuffer.allocate(48);

  int bytesRead = inChannel.read(buf);
  while (bytesRead != -1) {

    System.out.println("Read " + bytesRead);
    buf.flip();

    while(buf.hasRemaining()){
        System.out.print((char) buf.get());
    }

    buf.clear();
    bytesRead = inChannel.read(buf);
  }
  aFile.close();
```

上面代码，先是创建一个随机读取文件，并创建出通道。

然后通道的read方法把文件数据读入到缓冲区中。然后while语句里则是分批次读出buffer里的内容，每次读完一次以后都要进行flip模式反转，然后读出来以后清空这一批内容，继续read下一批文件数据，直到整个文件的内容被读完。


### Channel to Channel Transfers通道传输接口
在Java NIO中如果一个channel是FileChannel类型的，那么他可以直接把数据传输到另一个channel。这个特性得益于FileChannel包含的transferTo和transferFrom两个方法。

#### transferFrom()
FileChannel.transferFrom方法把数据从通道源传输到FileChannel：
``` java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

toChannel.transferFrom(fromChannel, position, count);
```

transferFrom的参数position和count表示目标文件的写入位置和最多写入的数据量。如果通道源的数据小于count那么就传实际有的数据量。 另外，有些SocketChannel的实现在传输时只会传输哪些处于就绪状态的数据，即使SocketChannel后续会有更多可用数据。因此，这个传输过程可能不会传输整个的数据。

#### transferTo()
transferTo方法把FileChannel数据传输到另一个channel,下面是案例：

``` java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);
```
这段代码和之前介绍transfer时的代码非常相似，区别只在于调用方法的是哪个FileChannel.

SocketChannel的问题也存在与transferTo.SocketChannel的实现可能只在发送的buffer填充满后才发送，并结束。

###  FileChannel
Java NIO中的FileChannel是用于连接文件的通道。通过文件通道可以读、写文件的数据。Java NIO的FileChannel是相对标准Java IO API的可选接口。

FileChannel不可以设置为非阻塞模式，他只能在阻塞模式下运行。

在使用FileChannel前必须打开通道，打开一个文件通道需要通过输入/输出流或者RandomAccessFile，下面是通过RandomAccessFile打开文件通道的案例：

``` java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
```

从文件通道内读取数据（Reading Data from a FileChannel）
``` java
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
```

写数据
``` java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}
```

注意，这里如果buf.put的数据量超过分配给buffer的量，则会抛出bufferOverFlow的错误。

操作完毕后，需要把通道关闭：
``` java
channel.close();
```
#### FileChannel Force
force方法会把所有未写磁盘的数据都强制写入磁盘。这是因为在操作系统中出于性能考虑回把数据放入缓冲区，所以不能保证数据在调用write写入文件通道后就及时写到磁盘上了，除非手动调用force方法。 force方法需要一个布尔参数，代表是否把meta data也一并强制写入。
``` java
channel.force(true);
```
