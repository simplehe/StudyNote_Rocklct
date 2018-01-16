# 网络编程基础

## 基本接口
Linux下，有一些基本的网络编程接口。

 - socket(), 创建Socket描述符，以供其他接口调用。
 - bind(): 通过socket接口创建的Socket描述符只存在于其协议簇空间，bind用来将一组固定的地址绑定到Socket描述符上。
 - listen():监听某个socket接口。
 - accept() 服务器方面，由内核产生一个Socket描述符用来，用来和TCP对端进行连接。
 - recv():接受对端发送的数据，从接收缓冲区复制数据。
 - connect() 创建与TCP服务器之间的Socket连接
 - send() 向对端发送数据，操作系统内核会将用户数据复制到**TCP套接字的发送缓冲区**，然后发送出去。

### 非阻塞的服务器程序
在常规的listen-accept模式下，基本都是阻塞blocked的。

**一个非常可用的方案是配合多线程+线程池管理来进行socket连接。** 但是显然，这不能解决所有问题，创建多余的线程也会给系统带来压力。

参考操作系统部分，5种IO模型笔记，我们知道IO有非阻塞的方法。

非阻塞接口的特点就是，被调用以后立即返回，使用如下函数可以将某句柄fd(文件描述符)设为非阻塞状态。

`fcntl(fd,F_SETFL,O_NONBLOCK)`

启用了这个函数以后，`recv()`接口在调用完成后会立即返回一个值：

1. recv()返回0，连接已经断开
2. recv()返回值大于0，接收数据完毕，且返回值是接受到的字节数
2. recv()返回-1,且errno不等于EAGAIN，操作遇到系统错误errno
3.recv()返回-1且erroo等于EAGAIN，操作还没完成。


如此一来，我们便可以用轮询的方式(对应5种IO模型中的第二种)，循环调用recv，但是这种方法会大幅度提高CPU占用率。所以操作系统为我们提供了更好的借口select()

### 基于select()接口的事件驱动服务器模型
大部分Linux函数都支持select函数，根据5种IO模型的笔记，当我们调用select函数时，线程自身会被阻塞，然后操作系统内核会替代我们轮询各个connection，探测文件句柄的状态变化。这也就是笔记中的**IO复用模型**

当我们同时要处理多个套接字的时候，也必须要使用IO复用。

下面是select接口的原型：

``` c++
FD_ZERO(int fd, fd_set* fds);
FD_SET(int fd,fd_set* fds);
FD_ISSET(int fd,fd_set* fds);
FD_CLR(int fd,fd_set* fds);
int select(int nfds,fd_set *readfds,fd_set *writefds,fd_set *exceptfds,struct timeval *timeout)
```


这里，fd_set可以简单地理解为按bit位标记句柄的标记数组，例如在fd_set中标记一个值为16的句柄，则fd_set的第16个bit位被标记为1.(zwlj：实际上就是用位运算优化标记数组)


所以调用select函数，相当于传递进去个标记数组，如果readfs中标记了16号句柄，那select将会检测(内核轮询)16号句柄是否可读。在select返回后，可以检查readfs来判断可读事件是否发生(数据是否复制到内核了)，如果可读，将会调用recv复制数据(此时recv一定能读到数据)。此外，用户还可以设置timeout时间。

FD_SET用来设置句柄的标记，FD_ISSET用来检查哪些句柄发生了事件(zwlj：哪些文件描述符指向的内存区域已经准备好读写数据了)

一下看select使用范例：

``` c
//str_cli
void
str_cli(FILE *fp, int sockfd)
{
    int         maxfdp1;
    fd_set      rset;
    char        sendline[MAXLINE], recvline[MAXLINE];

    FD_ZERO(&rset);
    for ( ; ; ) {
        FD_SET(fileno(fp), &rset);
        FD_SET(sockfd, &rset);
        maxfdp1 = max(fileno(fp), sockfd) + 1;
        Select(maxfdp1, &rset, NULL, NULL, NULL);

        if (FD_ISSET(sockfd, &rset)) {  /* socket is readable */
            if (Readline(sockfd, recvline, MAXLINE) == 0)
                err_quit("str_cli: server terminated prematurely");
            Fputs(recvline, stdout);
        }

        if (FD_ISSET(fileno(fp), &rset)) {  /* input is readable */
            if (Fgets(sendline, MAXLINE, fp) == NULL)
                return;     /* all done */
            Writen(sockfd, sendline, strlen(sendline));
        }
    }
}
```

如上代码，定义了一个函数str_cli，接受一个文件指针，和一个socket 文件描述符，根据Unix一切皆文件的思想，我们只要拿到文件描述符，就可以去读写文件描述符指向的内存，即IO.所以我们看到下面代码，先用FD_ZERO将标记数组清零，然后用一个永久循环，每次循环在rset结构中标记要读取的fd，然后用select去让操作系统轮询，一旦发现有地方可以读，就解开阻塞，交给下面的if语句去读取。


**select并不是“事件驱动”的最好选择，因为轮询探测句柄消耗了大量的事件。**

### 基于poll()
基于poll的模型跟select也是大同小异，都是**基于IO复用模型**。

select是使用一个fd_set的结构，这个结构是一个位图(bit标记数组),而fdset则是用一个结构体指针来实现

``` c
int poll (struct pollfd *fds, unsigned int nfds, int timeout);

struct pollfd {
    int fd; /* file descriptor */
    short events; /* requested events to watch */
    short revents; /* returned events witnessed */
};

```

上面函数分别函数和结构体的定义，select是基于三个位图实现的，而poll则是基于一个结构体指针了。

poll本质上和select没有区别，**它将用户传入的数组拷贝到内核空间**，然后查询每个fd对应的设备状态，如果设备就绪则在设备等待队列中加入一项并继续遍历，如果遍历完所有fd后没有发现就绪设备，则挂起当前进程，直到设备就绪或者主动超时，被唤醒后它又要再次遍历fd。这个过程经历了多次无谓的遍历。

它没有最大连接数的限制，原因是它是基于链表来存储的，但是**同样**有一个缺点：

 - **大量的fd的数组被整体复制于用户态和内核地址空间之间，而不管这样的复制是不是有意义**。
 - poll还有一个特点是“水平触发”，如果报告了fd后，没有被处理，**那么下次poll时会再次报告该fd**。


pollfd结构包含了要监视的event和发生的event，不再使用select“参数-值”传递的方式。同时，pollfd并没有最大数量限制（但是数量过大后性能也是会下降）。 和select函数一样，poll返回后，需要轮询pollfd来获取就绪的描述符。

从上面看，select和poll都需要在返回后，通过遍历文件描述符来获取已经就绪的socket。事实上，同时连接的大量客户端在一时刻可能只有很少的处于就绪状态，**因此随着监视的描述符数量的增长，其效率也会线性下降**。

### 基于epoll实现的异步事件模型通知
epoll当然也是基于IO复用模型的，epoll是在2.6内核中提出的，是之前的select和poll的增强版本。相对于select和poll来说，**epoll更加灵活，没有描述符限制**。

epoll使用一个文件描述符管理多个描述符，**将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次**。

epoll最大的好处在于它不会随着监听fd数目的增长而降低效率。因为在内核中的select实现中，它是采用轮询来处理的，轮询的fd数目越多，自然耗时越多(并且select还有数量限制)。

先来看看三个epoll的函数接口：

#### epoll_create

``` c
int epoll_create(int size);
```

新版本的linux中，epoll_create接口已经没什么用，一般调用epoll_create(0)来处理。**该函数会返回一个epoll专用的文件描述符，用于操作epoll的行为**。

size参数用来告诉内核这个监听的数目一共有多大。这个参数不同于select()中的第一个参数，给出最大监听的fd+1的值。需要注意的是，当创建好epoll句柄后，它就是会占用一个fd值，在linux下如果查看/proc/进程id/fd/，是能够看到这个fd的，所以在使用完epoll后，必须调用close()关闭，否则可能导致fd被耗尽。

这就可知epoll仅仅使用一个文件描述符来管理多个文件描述符。

#### epoll_ctl

``` c++
 int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

epoll的**事件注册函数**，它不同与select()是在监听事件时告诉内核要监听什么类型的事件，而是在这里先注册要监听的事件类型。

第一个参数就是epoll_create返回的那个管理fd，第二个参数op表示要执行的操作，可能值有以下三个宏：

 - EPOLL_CTL_ADD:注册新的fd到epfd中
 - EPOLL_CTL_MOD:修改已经注册的fd监听事件
 - EPOLL_CTL_DEL:从epfd中删除一个fd

第三个参数是需要监听的fd，第四个参数是告诉内核需要监听什么事，struct epoll_event结构如下：

``` c
typedef union epoll_data {
    void *ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
} epoll_data_t;

struct epoll_event {
    __uint32_t events; /* Epoll events */
    epoll_data_t data; /* User data variable */
};
```

先看看struct第一个整形参数events,表示事件类型，可以是以下几个宏的集合：

 - EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
 - EPOLLOUT：表示对应的文件描述符可以写；
 - EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
 - EPOLLERR：表示对应的文件描述符发生错误；
 - EPOLLHUP：表示对应的文件描述符被挂断；
 - EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
 - EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里

#### epoll_wait

``` c
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

等待事件的产生，类似于select()调用。参数events用来从内核得到事件的集合，maxevents告之内核这个events有多大，这个 maxevents的值不能大于创建epoll_create()时的size，参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。

所以我们仍然需要不停的循环去调用epoll_wait去等待用户中的请求，当用户请求到来的时候再去处理用户的事件。

#### epoll模式

epoll对文件描述符的操作有以下两种模式：

 - LT(LevelTriggered)水平触发
 - ET(EdgeTriggered)边缘触发

LT是默认，只要缓冲区有数据还没处理就会一直通知(返回socket)。所以这种条件下，epoll相当于一个高效的poll。也就是说应用程序收到epoll通知之后，可以**不立即处理事件**，因为之后还会再通知。

ET模式仅当状态发生变化的时候才获得通知(通知一次),这里所谓的状态的变化并不包括缓冲区中还有未处理的数据,也就是说,如果要采用ET模式,需要一直read/write直到出错为止。也就是说说边缘触发模式下，**程序必须立即处理事件**。

下面给出示例读数据代码：

``` c
n = 0;

while((nread = read(fd,buf + n,BUFSIZ-1))>0){
  n += nread;
}

if(nread == -1 && erron != EAGAIN){
  perror("read error");
}

```
