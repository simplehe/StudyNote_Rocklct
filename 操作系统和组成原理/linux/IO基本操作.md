## 基本IO操作

### read函数
位于头文件`#include <unistd.h>`

``` c
ssize_t read(int fd, void *buf, size_t count);
```

fd是要读取的文件描述符，对应内核的一块区域。buf是读取出数据的存放点(指针),count是希望读取的字节数。read的返回值往往低于count，因为经常还没读取到count就到文件末尾了。read返回值是实际读取的字节数。

所以count经常会被设置为buffer区域的字节大小减一。

从终端设备读，通常以行为单位，读到换行符就返回了。

### write函数
位于头文件`#include <unistd.h>`

``` c
ssize_t write(int fd, const void *buf, size_t count);
```

返回值：成功返回写入的字节数，出错返回-1并设置errno

写常规文件时，write的返回值通常等于请求写的字节数
count，而向终端设备或网络写则不一定。

blocking的write只有在缓冲区足以放下整个buffer时才返回（与blocking read并不相同）

nonblock write则是返回能够放下的字节数，之后调用则返回-1（errno = EAGAIN或EWOULDBLOCK）
