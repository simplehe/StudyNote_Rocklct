## servletconfig的作用
Servlet的声明周期init中，有init(ServletConfig)这个重载的方法，也就是说Servlet在创建的时候，会被传入一个ServletConfig对象，这个对象，就代表了项目中web.xml配置的信息。

这样做的好处是：如果将数据库信息、编码方式等配置信息放在web.xml中，如果以后数据库的用户名、密码改变了，则直接很方便地修改web.xml就行了，避免了直接修改源代码的麻烦。


 - String getServletName()  -- 获取当前Servlet在web.xml中配置的名字
 - String getInitParameter(String name) -- 获取当前Servlet指定名称的初始化参数的值
 - Enumeration getInitParameterNames()  -- 获取当前Servlet所有初始化参数的名字组成的枚举
 - ServletContext getServletContext()  -- 获取代表当前web应用的ServletContext对象


### ServletContext
WEB容器在启动时，它会为**每个WEB应用程序**都创建一个对应的ServletContext对象，它代表当前这整个web应用(整个大网站)。

ServletConfig对象中维护了ServletContext对象的引用，开发人员在编写servlet时，可以通过ServletConfig.getServletContext方法获得ServletContext对象。

由于一个WEB应用中的所有Servlet共享同一个ServletContext对象，因此**Servlet对象之间可以通过ServletContext对象来实现通讯**。ServletContext对象通常也被称之为context域对象。
