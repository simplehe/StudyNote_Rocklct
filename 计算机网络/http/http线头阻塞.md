## Head-of-Line Blocking （线头阻塞）
队列的首个packet由于它的目的端口正忙而被**延迟转发**，导致后面的packets被blocked

“线端阻塞（HOLB）”是一个存在已久的网络问题，其问题程度可与车管所办理相关业务相比，人们必须严格按照次序办事。“线端”的人是当前正在办理的那个人，可能会阻塞整个等待的队伍。当一个需要按顺序处理数据包的队列，加上各种不同的处理时间、网络延迟或数据包丢失时，这种问题为HOLB。

由于FIFO（先进先出）队列机制造成的，每个cross bar 输入端的FIFO首先处理的是在队列中最靠前的数据，而这时队列后面的数据对应的出口缓存可能已经空闲，但因为得不到处理而只能等待，这样既浪费了带宽又降低了系统性能。这就如同你在只有一条行车路线的马路上右转，但你前面有直行车，虽然这时右行线已经空闲，但你也只能等待。

### http导致线头阻塞
浏览器允许http对同一个域名发出几个并行连接。在某一个连接中(长连接非pipeline)，一个request必须发出然后接到一个response之后才发出下一个request。

这样后面的request必须要客户端接受到前面的response才能发出，这就是一种线头阻塞HOLB。head-of-line blocking problem。


### 管线化(pipelining)技术导致的线头阻塞问题
由于使用了pipelining技术，复用了同一个TCP connection。多个request在同一个tcp连接里发出，**然而，response还是需要在一个tcp连接里按顺序发回来**。

这里就出现问题了，因为response规定必须按顺序发回来，所以假如我按序发出RequestA和RequestB。假如A的 response的contentlength非常大，responseB则非常小，那第二个response相当于被第一个response给阻塞住不能发出了。这就导致了HOLB，head-of-line blocking problem。

所以pipeline技术还是没有解决这个HOLB问题，所以导致性能提升不明显而没有被广泛采用。
