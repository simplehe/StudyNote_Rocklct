## JSP
JSP全称Java Server Pages，是一种动态网页开发技术。它使用JSP标签在HTML网页中插入Java代码。标签通常以<%开头以%>结束。

**JSP是一种Java servlet**，主要用于实现Java web应用程序的用户界面部分。网页开发者们通过结合HTML代码、XHTML代码、XML元素以及嵌入JSP操作和命令来编写JSP。

### JSP对比CGI

JSP程序与(其他)CGI程序(概念见网络部分笔记)有着相似的功能，但和CGI程序相比，JSP程序有如下优势：
 - 性能更加优越，因为JSP可以直接在HTML网页中动态嵌入元素而不需要单独引用CGI文件。
 - 服务器调用的是已经编译好的JSP文件，而不像CGI/Perl那样必须先载入解释器和目标脚本。
 - JSP **基于Java Servlet API**，因此，JSP拥有各种强大的企业级Java API，包括JDBC，JNDI，EJB，JAXP等等。
 - JSP页面可以与处理业务逻辑的 Servlet 一起使用，这种模式被Java servlet 模板引擎所支持。

 最后，JSP是Java EE不可或缺的一部分，是一个完整的企业级应用平台。这意味着JSP可以用最简单的方式来实现最复杂的应用。

 ### JSP 处理
 以下步骤表明了Web服务器是如何使用JSP来创建网页的：

 - 就像其他普通的网页一样，您的浏览器发送一个HTTP请求给服务器。
 - Web服务器识别出这是一个对JSP网页的请求，并且将该请求**传递给JSP引擎**。通过使用URL或者.jsp文件来完成。
 - JSP引擎从磁盘中载入JSP文件，然后将它们**转化为servlet**。这种转化只是简单地将**所有模板文本改用println()语句**，并且将所有的JSP元素转化成Java代码。
 - JSP引擎将servlet编译成可执行类，并且将**原始请求传递给servlet引擎**。
 - Web服务器的某组件将会调用servlet引擎，然后载入并执行servlet类。在执行过程中，servlet产生HTML格式的输出并将其内嵌于HTTP response中上交给Web服务器。
 - Web服务器以静态HTML网页的形式将HTTP response返回到您的浏览器中。
 - 最终，Web浏览器处理HTTP response中动态产生的HTML网页，就好像在处理静态网页一样。

![](image/jsp0.jpg)

一般情况下，JSP引擎会检查JSP文件对应的servlet是否已经存在，并且检查JSP文件的修改日期是否早于servlet。如果JSP文件的修改日期早于对应的servlet，那么容器就可以确定JSP文件没有被修改过并且servlet有效。这使得整个流程与其他脚本语言（比如PHP）相比要高效快捷一些。

总的来说，**JSP网页就是用另一种方式来编写servlet而不用成为Java编程高手。除了解释阶段外，JSP网页几乎可以被当成一个普通的servlet来对待**。
