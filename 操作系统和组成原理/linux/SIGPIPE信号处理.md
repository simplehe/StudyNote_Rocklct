## SIGPIPE信号处理
当服务器close一个连接时，若client端接着发数据。根据TCP协议的规定，会收到一个RST响应，client再往这个服务器发送数据时，系统会发出一个SIGPIPE信号给进程，告诉进程这个连接已经断开了，不要再写了。

也就是说， 对一个对端已经关闭的socket调用两次write, 第二次将会生成SIGPIPE信号, 该信号默认结束进程.

在linux下写socket的程序的时候，如果尝试send到一个disconnected socket上，就会让底层抛出一个SIGPIPE信号。这个信号的缺省处理方法是退出进程，大多数时候这都不是我们期望的。因此我们需要重载这个信号的处理方法。调用以下代码，即可安全的屏蔽SIGPIPE：

``` c
signal （SIGPIPE， SIG_IGN）；
```

我的程序产生这个信号的原因是:
client端通过 pipe 发送信息到server端后，就关闭client端, 这时server端,返回信息给 client 端时就产生Broken pipe 信号了，服务器就会被系统结束了。
