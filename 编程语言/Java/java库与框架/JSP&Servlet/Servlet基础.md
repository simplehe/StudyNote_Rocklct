## servlet基础
Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。

使用 Servlet，您可以收集来自网页表单的用户输入，呈现来自数据库或者其他源的记录，还可以动态创建网页。

![](image/servlet0.jpg)


**Java Servlet 通常情况下与使用 CGI（Common Gateway Interface，公共网关接口）(概念见网络部分笔记)实现的程序可以达到异曲同工的效果**。但是相比于 CGI，Servlet 有以下几点优势：

 - 性能明显更好。
 - **Servlet 在 Web 服务器的地址空间内执行**。这样它就没有必要再创建一个单独的进程来处理每个客户端请求。
 - Servlet 是独立于平台的，因为它们是用 Java 编写的。
 - 服务器上的 Java 安全管理器执行了一系列限制，以保护服务器计算机上的资源。因此，Servlet 是可信的。
 - Java 类库的全部功能对 Servlet 来说都是可用的。它可以通过 sockets 和 RMI 机制与 applets、数据库或其他软件进行交互。

### servlet和cgi的区别

当用户浏览器发出一个Http/CGI的请求，或者说调用一个CGI程序的时候，服务器端就要**新启用一个进程(而且是每次都要调用)**，调用CGI程序越多(特别是访问量高的时候)，就要消耗系统越多的处理时间，只剩下越来越少的系统资源

而Servlet充分发挥了服务器端的资源并高效的利用。每次调用Servlet时并不是新启用一个进程，而是在一个Web服务器的进程(同一个进程)中**共享和分离线程**。对于客户端的请求，只需要创建Servlet的实例一次，因此节省了大量的内存资源。Servlet在初始化后就保留在内存中，因此每次作出请求时无需加载。

zwlj:所以本质区别便是，servlet处于服务器进程中，它通过**多线程方式**运行其service方法，一个实例可以服务于多个请求，并且其实例一般不会销毁，而CGI对每个请求都产生新的进程，服务完成后就销毁，所以效率上低于servlet。

### Servlet包
Java Servlet 是**运行在带有支持 Java Servlet 规范**的解释器的 web 服务器上的 Java 类。

Servlet 可以使用 javax.servlet 和 javax.servlet.http 包创建，它是 Java 企业版的标准组成部分，Java 企业版是支持大型开发项目的 Java 类库的扩展版本。

这些类实现 Java Servlet 和 JSP 规范。
