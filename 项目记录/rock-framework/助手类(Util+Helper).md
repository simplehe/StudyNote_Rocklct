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

#### AopHelper
这个类用来实现Aop的核心功能。首先通过某个具体的切面标注实例(通过对用户编写的切面代理类反射获得的，可以知道用户想包装什么样的类，比如想这个切面包装所有Controller类),获取到所有应该被代理的Class类(也要判断被标注的不是Aspect初始类)(createTargetClassSet方法)。

createProxyMap这个方法用于将切面类(封装了增强逻辑)和需要背标注的那些具体类，两者用Map映射起来。

createTargetMap则是利用了createProxyMap这个方法，我们用之前的方法拿到了具体代理类Class和它对应处理的Class Set的映射Map。那么显然我们现在需要做的就是反过来了，我们需要得到每个需要被切面封装处理的Class，他一共对应了哪些切面，这样我们才可以封装出一个proxyChain。

##### 缕清逻辑
首先我们知道Aspect是一个标注类，用来标注切面类，切面类封装了很多增强逻辑，我们会用切面类去封装一些具体的类。Aspect标注切面类的同时，会传递一个value，这个value会表明，需要被包装的类是什么类型，而这个value当然也是一个标注类的类型，比如我们写的某个切面类需要包装所有controller，那我们当然会用标注Aspect(Controller.class)，去标注，这样就会知道，**这个切面是去包装"被controller标注的类"**。这里还要注意，用户写的切面类都必须要继承AspectProxy这个模板类，还必须带有Aspect注解。

而AopHelper就是用于映射每个Aspect切面类和需要被它包装的所有类。所以我们在createProxyMap中，先找出所有用户写的继承了AspectProxy的切面类，然后得到这些切面类的标注，看看它们分别想标注什么样的类，然后返回一个set集合和这些切面类对应起来。

最后我们利用createTargetMap方法，反向操作，得到每个具体需要被切面包装的类，比如某个controller类，会有哪些AspectProxy处理它，这样得到一个反向映射集合。很显然这个集合才是我们想要的，因为只有这样，我们在执行某个controller类的时候，才会取得一个proxyChain。**要注意createTargetMap这个方法，它封装的是某个具体要被处理的Class，和处理它的AspectProxy实例**。也就是说在这里，实例已经被封装进去了，用以之后封装成proxyChain。


静态块正式遵循以上逻辑，先获取proxyMap，获得所有代理Class及其对应的Bean Class，最后反向操作，获得每个Bean Class及其对应的ProxyList。最后将其一一set到BeanHelper的Bean Map当中
