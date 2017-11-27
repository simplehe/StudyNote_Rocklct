## 助手类总结
在package com.rocklct.framework下，汇集几个工具助手类。


### util包
其中

 - ConfigConstant(常量接口，让它维护用户配置文件\[rocklct.properties\]中相关的配置项名称)


util包下
 - PropsUtil类用来导入配置文件
 - CollectionUtil,StringUtil对相关类功能做了一下封装，利用了Collections框架。
 - ClassUtil用来协助加载某个包下的类

 **ClassUtil的原理**大致是，输入包名，转换为路径，访问该路径下的所有文件，如果是class就直接读取并且加载。如果是目录(子包)，就递归下去读class文件，如果是jar包，则是进入里面读jar包下的class文件。

 - ReflectionUtil

 **ReflectionUtil类**用来实现一些，拿到具体的Class类后反射执行方法。比如用Class类创建一个实例，调用一个方法，设置一个变量值。对这些功能做了一下封装。

  -  CodecUtil

**CodecUtil类**，是一个url的解码(decode)和编码(encode)类。里面封装了url类的解码和编码的方法，用UTF-8编码作为标准。在网页传输是，url的一些符号经常会被解析成%xy这样的形式，我们必须把这些%xy形式的字符还原成原字符。这样我们可以依靠UrlDecoder这个类。中途操作需要把字符串(内存中保存为Unicode)转成bytes进行操作，所以需要用UTF-8解码这个url成bytes数组，然后操作完以后，再根据UTF-8编码集转成Unicode流(也就是字符串)

-  StreamUtil

**StreamUtil类**，这个类就封装了一个方法，就是把输入流(inputStream)读取并转换成字符串。

**JsonUtil类**，这个类用以将POJO(Plain Ordinary Java Object),也就是简单java对象，转换成json字符串。中间用到了Jackson库。

### helper包


helper包下
#### ConfigHelper类
用来读取文件配置(利用了PropsUtil)

#### ClassHelper类
ClassHelper类(利用ClassUtil)，用来读取应用基础包路径下的类，并且抽取出带有注解的类并放入集合中。Service和Controller标注的类，作为框架管理的Bean类。

经过切面功能扩展以后，新增两个方法，一个是获得继承了某个具体父类的所有子类Class，一个是获得被某个特定标注标记的所有Class。

#### BeanHelper类
相当于一个Bean容器，这个类的核心就是内置一个Map，我们首先会根据ClassHelper去获取应用中进行了标注的Class，然后再把这些BeanClass(Service和Controller类)一一实例化，实例化以后放入到BeanHelper的这个Map中。Map的key设置为Class类，value为这个具体Class的实例。里面还有setBean方法，用于面向切面编程，controller类的实例被包装成代理类实例后，要重新装载进核心map中。

#### IocHelper类
用来实现依赖注入的功能。充当IoC容器，原理很直接，在静态块中先利用BeanHelper获取到BeanMap，这个Map的key值是Service和Controller类和相应的实例。获取Bean实例，查看里面的成员变量是否有Inject注解，有的话利用ReflectionUtil去为这个成员变量注入实例值。不过这里要注意，注入的值依旧是从Map里获取Service实例(因为也属于Bean)，而不是重新创建。从这个角度来说，里面的所有对象都是单例。

#### ControllerHelper类
这个类的核心，也是一个Map，这个Map映射request和handler两个Bean对象。也就是说，当客户端请求一个地址的时候，这个ControllerHelper能帮助你根据地址找到对应的handler来处理。
