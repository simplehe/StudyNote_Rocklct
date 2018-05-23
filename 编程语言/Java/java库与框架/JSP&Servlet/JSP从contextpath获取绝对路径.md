## JSP取得路径


### 写相对路径
 JSP中究竟采用绝对路径还是采用相对路径随着所采用技术的越来越复杂，这个问题也变得越来越难以解决。

 相对路径固然比较灵活，但如果想复制页面内的代码却变得比较困难，**因为不同的页面具有不同的相对路径(各种./index还有.../index)，每个网页所在的层次不一样**，复制后必须修改每一个连接的路径。
如果页面被多于一个的页面所包含，那么被包含页面中的相对路径将是不正确的。

### 写绝对路径
写绝对路径的话，就有一个不好的地方，那就是应用部署之后名字可能会不一样。

如Web应用发布为MyApp，则路径”/MyApp/images/title.gif”是正确的，但发布为另一应用时如MyApp2，这个路径就不对了，也许这个情况比较少，但以default方式发布Web应用时以上绝对路径也不同。

### ${pageContext.request.contextPath}
这个方式挺重要的，这个contextPath可以获得项目的根路径。用了这种方式之后，就可以方便的用绝对路径了。

如原路径为：
\/images\/title.gif

改为 ${pageContext.request.contextPath}/images/title.gif

代码${pageContext.request.contextPath}的作用是取出部署的应用程序名，这样不管如何部署，所用路径都是正确的。
