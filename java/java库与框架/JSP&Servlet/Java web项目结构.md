## Java web项目结构的一些概念

![](image/webapp0.png)

如若用itelliJ创建java web项目，那么简单结构便是如下所示。

src目录用来放java源码，那么WEB-INF目录呢？

### WEB-INF
WEB-INF是Java的WEB应用的**安全目录**。所谓安全就是**客户端无法访问，只有服务端可以访问的目录**。

如果想在页面中直接访问其中的文件， **必须通过web.xml文件**对要访问的文件进行相应映射才能访问。

**web.xml的作用：Web应用程序配置文件，描述了 servlet 和其他的应用组件配置及命名规则。**

Web-Inf文件夹中除了有**classes文件夹和一个web.xml文件**外、还有lib文件夹（用于存放需要的jar包）（用于配置，比如说用来配置过滤器等。）。classes文件夹用于存放编译后输出的class文件。

#### web.xml
一般的web工程中都会用到web.xml，web.xml主要用来配置，可以方便的开发web工程。web.xml主要用来配置Filter、Listener、Servlet等。但是要说明的是web.xml并不是必须的，一个web工程可以没有web.xml文件。
