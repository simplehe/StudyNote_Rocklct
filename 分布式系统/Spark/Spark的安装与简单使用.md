## Spark安装
Spark是用Scala写的，运行在JVM上。所以运行环境要Java1.7以上，如果用PySpark，则要安装python2.6或者3.4+。Spark1.6.2对应Scala2.10，Spark2.0对应Scala2.11

### 下载
官网下载linux对应的tgz包，然后再linux环境下解压即可。

![](image/spark0.jpg)


### Spark shell
spark在linux下解压以后，进入bin目录，可以执行spark shell。

 - Spark的shell帮助用户处理分布在集群中的数据
 - Spark把数据加载到节点的内存中，因此分布式处理可在秒级完成。
 - 使用shell可以**分析数据，快速迭代计算，实时查询**。
 - 有**Python shell和Scala shell**两种

pySpark在bin目录下，运行pyspark来执行。Scala shell则是运行spark-shell来运行。

在shell下，我们就可以读入文件到内存，然后操作rdd。
