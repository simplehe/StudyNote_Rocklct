## Spark RDD

http://blog.csdn.net/guohecang/article/details/51736572

 在集群背后，有一个非常重要的分布式数据架构，即**弹性分布式数据集**（Resilient Distributed Dataset，RDD）。RDD是Spark的最基本抽象,是对分布式内存的抽象使用，**实现了以操作本地集合的方式来操作分布式数据集的抽象实现**。

RDD是Spark最核心的东西，**它表示已被分区，不可变的并能够被并行操作的数据集合**，不同的数据集格式对应不同的RDD实现。**RDD必须是可序列化的**。RDD可以cache到内存中，每次对RDD数据集的操作之后的结果，都可以存放到内存中，下一个操作可以直接从内存中输入，**省去了MapReduce大量的磁盘IO操作**。这对于迭代运算比较常见的机器学习算法, 交互式数据挖掘来说，效率提升比较大。

### RDD特点
1. 创建：只能通过转换 ( transformation ，如**map/filter/groupBy/join** 等，区别于动作 action) 从两种数据源中创建 RDD 1 ）稳定存储中的数据； 2 ）其他 RDD。

2. 只读：状态不可变，不能修改。

3. 分区：支持使 RDD 中的元素**根据那个 key 来分区** ( partitioning ) ，保存到多个结点上。还原时只会重新计算丢失分区的数据，而不会影响整个系统。

4. 路径：在 RDD 中叫世族或血统 ( lineage ) ，即 RDD 有充足的信息关于它是如何从其他 RDD 产生而来的。

5. 持久化：支持将会被重用的 RDD 缓存 ( 如 in-memory 或溢出到磁盘 )。

6. 延迟计算： Spark 也会延迟计算 RDD ，使其能够将转换管道化 (pipeline transformation)。

7. 操作：丰富的转换（transformation）和动作 (action),count/reduce/collect/save 等。执行了多少次transformation操作，RDD都不会真正执行运算（记录lineage），只有当action操作被执行时，运算才会触发。

### RDD优势
1. RDD只能从持久存储或通过Transformations操作产生，相比于分布式共享内存(DSM)可以更高效实现容错，对于丢失部分数据分区只需根据它的lineage就可重新计算出来，而不需要做特定的Checkpoint。
2. RDD的不变性，可以实现类Hadoop MapReduce的推测式执行。
3. RDD的数据分区特性，可以通过数据的本地性来提高性能，这不Hadoop MapReduce是一样的。
4. RDD都是可序列化的，**在内存不足时可自动降级为磁盘存储**，把RDD存储于磁盘上，这时性能会有大的下降但不会差于现在的MapReduce。
5. 批量操作：任务能够根据数据本地性 (data locality) 被分配，从而提高性能。
