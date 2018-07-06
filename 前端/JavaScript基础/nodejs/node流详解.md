## node流详解
node处理数据，有buffer模式和stream模式。buffer模式就是先读到内存再操作，显然数据大的时候就不行了，使用流模式是很有必要的。

在之前的buffer和stream笔记里已经介绍了两种对象

nodejs里面的stream一般分四种, 其中转换流是一种特殊的读写流：

 - 输入流(stream.Readable)
 - 输出流(stream.Writable)
 - 读写流(stream.Duplex)
 - 转换流(stream.Transform)

 另外, nodejs里面的流有两种模式, 二进制模式和对象模式.

 - 二进制模式, 每个分块都是buffer或者string对象.
 - 对象模式, 流内部处理的是一系列普通对象.
