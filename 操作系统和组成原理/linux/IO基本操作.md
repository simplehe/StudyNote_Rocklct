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


### lseek
每个打开的文件都有一个“**当前文件偏移量**”，是一个非负整数，用以度量从文件开始处计算的字节数。通常，读写操作都是从当前文件偏移量处开始，并使偏移量增加所读或写的字节数。默认情况下，你打开一个文件时(open)，除非指定 O_APPEND 参数，不然位移量被设为0。

需要包含的头文件：`<sys/types.h>`, `<unistd.h>` 函数原型：

``` c++
off_t lseek(int filesdes, off_t offset, int whence)
```

功能：设置文件内容读写位置

返回值：成功返回新的文件位移，出错返回-1；

同样 off_t 是系统头文件定义的数据类型，相当于 signed int


1. whence 是 SEEK_SET, 那么该文件的位移量设置为据文件开始处 offset 个字节

2. whence 是 SEEK_CUR, 那么该文件的位移量设置为当前值加 offset。offset 可为正或负

3. whence 是 SEEK_END, 那么该文件的位移量设置为文件长度加 offset。offset 可为正或负
