## Bean类
框架封装了一些Bean类型，放在com.rocklct.bean包内。

### Request类

里面有requestMethod字段和requestPath字段。

一个代表请求的方式(get or post),一个代表请求的路径。

### Handler类
即是处理器类，当我们给出一个request method(get或者post)，和要访问的地址的时候。就会去找到这个相应的处理器。所以处理器Bean类里的两个字段是ControllerClass和对应的Method。具体的Handler类Bean将会用ControllerHelper来进行查找。根据对应地址，从Helper类里找到对应的Handler以后，**handler会根据哪一个controller里的哪一个method，去invoke这个method来做出回应。**


### Param类
Param这个Bean类实质上是对Http请求中的参数进行了一个封装。比如用户get或者post，传递来一些参数，我们就会从HttpServletRequest对象中获取，并且封装成我们的Param的bean类。其核心也是维护一个Map，里面可以根据String类型的键值来寻找对应的value值。


### View类
View这个Bean类，就是试图Bean类。当用户使用web框架的时候，写Action方法的返回类型就是一个View，View就是对Jsp页面进行了一个封装。所以View里首先由path路径，还有model数据。model数据是一个map，用来存取请求给某个view页面时，post或者get传输过来的数据map(json).

### Data类
Data这个Bean类，和View类一样，也是Action的返回值的一种。当返回值为View类的时候是渲染jsp页面，当返回值是Data类的时候，返回的是json数据。Data类里面仅只封装一个Object，框架会将这个对象用Jackson解析成json字符串，然后写入HttpServletResponse对象中，从而直接输出到浏览器。
