## OOM
OutOfMemoryError这个错误大部分开发人员都有遇到过，产生该错误的原因大都出于以下原因：JVM内存过小、程序不严密，产生了过多的垃圾。

导致OutOfMemoryError异常的常见原因有以下几种：
1. 内存中加载的数据量过于庞大，如一次从**数据库**取出过多数据；
2. 集合类中有对对象的引用，使用完后未清空，使得JVM不能回收；
3. 代码中存在死循环或循环产生过多重复的对象实体；
4. 使用的第三方软件中的BUG；
5. 启动参数内存值设定的过小；

下面来阐述OOM发生的情况和对应解决方法

### java.lang.OutOfMemoryError:Java heap space
触发java.lang.OutOfMemoryError: Java heap space最常见的原因就是应用程序需要的堆空间是XXL号的，但是JVM提供的却是S号。解决方法也很简单，提供更大的堆空间即可。

#### 解决方案
第一个解决方案是显而易见的，你应该确保有足够的堆空间来正常运行你的应用程序，在JVM的启动配置中增加如下配置：

```
-Xmx1024m
```


上面的配置分配1024M堆空间给你的应用程序，当然你也可以使用其他单位，比如用G表示GB，K表示KB。下面的示例都表示最大堆空间为1GB：
```
java -Xmx1073741824 com.mycompany.MyClass
java -Xmx1048576k com.mycompany.MyClass
java -Xmx1024m com.mycompany.MyClass
java -Xmx1g com.mycompany.MyClass
```

然后，更多的时候，单纯地增加堆空间不能解决所有的问题。如果你的程序存在内存泄漏，一味的增加堆空间也只是推迟java.lang.OutOfMemoryError: Java heap space错误出现的时间而已，并未解决这个隐患。除此之外，垃圾收集器在GC时，应用程序会停止运行直到GC完成，而增加堆空间也会导致GC时间延长，进而影响程序的吞吐量。

如果你想完全解决这个问题，那就好好提升自己的编程技能吧，当然运用好Debuggers, profilers, heap dump analyzers等工具，可以让你的程序最大程度的避免内存泄漏问题。

### java.lang.OutOfMemoryError:GC overhead limit exceeded
Java运行时环境（JRE）包含一个内置的垃圾回收进程，而在许多其他的编程语言中，开发者需要手动分配和释放内存。

默认情况下，当应用程序花费超过98%的时间用来做GC并且回收了不到2%的堆内存时，会抛出java.lang.OutOfMemoryError:GC overhead limit exceeded错误。具体的表现就是你的应用几乎耗尽所有可用内存，并且GC多次均未能清理干净。

java.lang.OutOfMemoryError:GC overhead limit exceeded错误是一个信号，示意你的应用程序在垃圾收集上花费了太多时间但却没有什么卵用。默认超过98%的时间用来做GC却回收了不到2%的内存时将会抛出此错误。那如果没有此限制会发生什么呢？GC进程将被重启，100%的CPU将用于GC，而没有CPU资源用于其他正常的工作。如果一个工作本来只需要几毫秒即可完成，现在却需要几分钟才能完成，我想这种结果谁都没有办法接受。

你可以通过在JVM添加选项来推迟这个错误，但是不建议，最好的方法还是研究代码优化其中的问题。

### java.lang.OutOfMemoryError:Permgen space

java.lang.OutOfMemoryError: PermGen space错误就表明永久代(方法区)所在区域的内存已被耗尽。

持久代主要存储的是每个类的信息，比如：类加载器引用、运行时常量池（所有常量、字段引用、方法引用、属性）、字段(Field)数据、方法(Method)数据、方法代码、方法字节码等等。我们可以推断出，PermGen的大小取决于被加载类的数量以及类的大小。

#### 解决方案

##### 解决初始化时的OutOfMemoryError
当在应用程序启动期间触发由于PermGen耗尽引起的OutOfMemoryError时，解决方案很简单。 应用程序需要更多的空间来加载所有的类到PermGen区域，所以我们只需要增加它的大小。 为此，请更改应用程序启动配置，并添加（或增加，如果存在）-XX：MaxPermSize参数，类似于以下示例：

```
java -XX:MaxPermSize=512m com.yourcompany.YourClass
```

##### 解决Redeploy时的OutOfMemoryError
分析dump文件：首先，找出引用在哪里被持有；其次，给你的web应用程序添加一个关闭的hook，或者在应用程序卸载后移除引用。

方案待补充。。。。

##### 解决运行时OutOfMemoryError
首先你需要检查是否允许GC从PermGen卸载类，JVM的标准配置相当保守，只要类一创建，即使已经没有实例引用它们，其仍将保留在内存中，特别是当应用程序需要动态创建大量的类但其生命周期并不长时，允许JVM卸载类对应用大有助益，你可以通过在启动脚本中添加以下配置参数来实现：

```
-XX:+CMSClassUnloadingEnabled
```

方法待补充。。。

### java.lang.OutOfMemoryError:Metaspace
前文已经提过，PermGen区域用于存储类的名称和字段，类的方法，方法的字节码，常量池，JIT优化等，但从Java8开始，Java中的内存模型发生了重大变化：引入了称为Metaspace的新内存区域，而删除了PermGen区域。请注意：不是简单的将PermGen区所存储的内容直接移到Metaspace区，PermGen区中的某些部分，已经移动到了普通堆里面。

Java8做出如此改变的原因包括但不限于：
 - 应用程序所需要的PermGen区大小很难预测，设置太小会触发PermGen OutOfMemoryError错误，过度设置导致资源浪费。
 - 提升GC性能，在HotSpot中的每个垃圾收集器需要专门的代码来处理存储在PermGen中的类的元数据信息。从PermGen分离类的元数据信息到Metaspace，由于Metaspace的分配具有和Java Heap相同的地址空间，因此Metaspace和Java Heap可以无缝的管理，而且简化了FullGC的过程，以至将来可以并行的对元数据信息进行垃圾收集，而没有GC暂停。
 - 支持进一步优化，比如：G1并发类的卸载，也算为将来做准备吧

#### 解决方案
第一个解决方案是显而易见的，既然应用程序会耗尽内存中的Metaspace区空间，那么应该增加其大小，更改启动配置增加如下参数：
```
// 告诉JVM：Metaspace允许增长到512，然后才能抛出异常
-XX：MaxMetaspaceSize = 512m
```

另一个方法就是删除此参数来完全解除对Metaspace大小的限制（默认是没有限制的）。默认情况下，对于64位服务器端JVM，MetaspaceSize默认大小是21M（初始限制值），一旦达到这个限制值，FullGC将被触发进行类卸载，并且这个限制值将会被重置，新的限制值依赖于Metaspace的剩余容量。如果没有足够空间被释放，这个限制值将会上升，反之亦然。在技术上Metaspace的尺寸可以增长到交换空间，而这个时候本地内存分配将会失败

### java.lang.OutOfMemoryError:Unable to create new native thread
出现java.lang.OutOfMemoryError:Unable to create new native thread就意味着Java应用程序已达到其可以启动线程数量的极限了。

当JVM向OS请求创建一个新线程时，而OS却无法创建新的native线程时就会抛出Unable to create new native thread错误。一台服务器可以创建的线程数依赖于物理配置和平台，建议运行下文中的示例代码来测试找出这些限制。总体上来说，抛出此错误会经过以下几个阶段：

 - 运行在JVM内的应用程序请求创建一个新的线程
 - JVM向OS请求创建一个新的native线程
 - OS尝试创建一个新的native线程，这时需要分配内存给新的线程
 - OS拒绝分配内存给线程，因为32位Java进程已经耗尽内存地址空间（2-4GB内存地址已被命中）或者OS的虚拟内存已经完全耗尽
 - Unable to create new native thread错误将被抛出

有时，你可以通过在OS级别增加线程数限制来绕过这个错误。如果你限制了JVM可在用户空间创建的线程数，那么你可以检查并增加这个限制：

当你的应用程序产生成千上万的线程，并抛出此异常，表示你的程序已经出现了很严重的编程错误，我不觉得应该通过修改参数来解决这个问题，不管是OS级别的参数还是JVM启动参数。更可取的办法是分析你的应用是否真的需要创建如此多的线程来完成任务？是否可以使用线程

### java.lang.OutOfMemoryError:Out of swap space?
Java应用程序在启动时会指定所需要的内存大小，可以通过-Xmx和其他类似的启动参数来指定。在JVM请求的总内存大于可用物理内存的情况下，操作系统会将内存中的数据交换到磁盘上去。

Out of swap space?表示交换空间也将耗尽，并且由于缺少物理内存和交换空间，再次尝试分配内存也将失败。

java.lang.OutOfMemoryError:Out of swap space?往往是由操作系统级别的问题引起的，例如：

 - 操作系统配置的交换空间不足。
 - 系统上的另一个进程消耗所有内存资源。

解决这个问题有几个办法，通常最简单的方法就是增加交换空间，不同平台实现的方式会有所不同

### java.lang.OutOfMemoryError:Requested array size exceeds VM limit

Java对应用程序可以分配的最大数组大小有限制。不同平台限制有所不同，但通常在1到21亿个元素之间。

当你遇到Requested array size exceeds VM limit错误时，意味着你的应用程序试图分配大于Java虚拟机可以支持的数组。

检查你的代码库，看看你是否真的需要这么大的数组。也许你可以减少数组的大小，或者将数组分成更小的数据块，然后分批处理数据。

### Out of memory:Kill process or sacrifice child
操作系统是建立在进程的概念之上，这些进程在内核中作业，其中有一个非常特殊的进程，名叫“内存杀手（Out of memory killer）”。当内核检测到系统内存不足时，OOM killer被激活，然后选择一个进程杀掉。哪一个进程这么倒霉呢？选择的算法和想法都很朴实：谁占用内存最多，谁就被干掉。

当可用虚拟虚拟内存(包括交换空间)消耗到让整个操作系统面临风险时，就会产生Out of memory:Kill process or sacrifice child错误。在这种情况下，OOM Killer会选择“流氓进程”（很可能是这个虚拟机进程）并杀死它。

解决这个问题最有效也是最直接的方法就是升级内存，其他方法诸如：调整OOM Killer配置、水平扩展应用，将内存的负载分摊到若干小实例上
