## jdk动态代理
关于代理模式，可参阅设计模式。这里着重讲讲java1.5后引入的java.lang.Proxy类。uml如下图

![](image/proxy0.png)

java.lang.reflect.Proxy：这是 Java 动态代理机制的主类，它提供了**一组静态方法**来为一组接口**动态地生成代理类及其对象**。

来观察一下Proxy的静态方法

``` java
// 方法 1: 该方法用于获取指定代理对象所关联的调用处理器
static InvocationHandler getInvocationHandler(Object proxy)

// 方法 2：该方法用于获取关联于指定类装载器和一组接口的动态代理类的类对象
static Class getProxyClass(ClassLoader loader, Class[] interfaces)

// 方法 3：该方法用于判断指定类对象是否是一个动态代理类
static boolean isProxyClass(Class cl)

// 方法 4：该方法用于为指定类装载器、一组接口及调用处理器生成动态代理类实例
static Object newProxyInstance(ClassLoader loader, Class[] interfaces,
    InvocationHandler h)
```

我们来观察一下这几个方法。

首先要说的是java.lang.reflect.InvocationHandler这个类。

这是调用处理器接口，它自定义了一个 invoke 方法，用于集中处理在动态代理类对象上的方法调用，通常在该方法中实现对委托类的代理访问。

怎么理解这个呢，其实InvocationHandler就是一个动态调用器。我们通过实现这个接口，来决定我们代理一个RealObject时要怎样执行。

比如

``` java

public class DynamicProxy implements InvocationHandler{
  private Object realSubject;

  public DynamicProxy(Object realSubject){
    this.realSubject = realSubject;
  }

  @Override
  public Object invoke(Object proxy,Method method,Object[] args) throws Throwable{

    doSomethingBefore();
    Object result = method.invoke(realSubject,args);
    doSomethingAfter();
  }
...

}


```

以上我们就实现了InvocationHandler接口，实现了这个invoke调用的方法。通过这么做，我们就可以对代理调用进行一层包装。在对realObject进行调用前后，我们可以自己进行了一些方法的调用。

需要注意这是一个通用的调用器，因为持有的真实类对象是realObject而不是具体某个AAA类.有了这个调用器，我们就能比较方便的动态创建这个xxxproxy实例了。

### 创建动态代理的步骤

多说无益，可以直接参考代码

``` java
RealSubject realSubject = new RealSubject();

// DynamicProxy实现了 InvocationHandler 接口，并能实现方法的分派转发
InvocationHandler dynamicProxy = new DynamicProxy(realSubject);

// 通过 Proxy 直接创建动态代理类实例
Subject proxy = (Subject)Proxy.newProxyInstance(
Subject.Class.getClassLoader(),
Subject.Class.getInterfaces(),
dynamicproxy
);
```

首先我们当然要先自己定义一个动态调用器，比如上文的dynamicProxy类，继承自InvocationHandler接口。然后我们创建这个动态调用器的实例,传入一个真实对象。

之后我们就可以根据jdk，Proxy类提供的newProxyInstance方法来进行创建，创建出一个动态的，xxxProxy实例。可想而知，我们必须告诉Proxy类一些参数，它才能帮助我们正确的创建，比如说，生成的这个动态实例对象(由设计模式，Proxy也继承Subject接口)具有什么方法啊(由第二个参数getInterfaces提供),还要告诉如何代理执行这个方法呢(由第三个参数调用器提供)。这样返回来的结果，只要经过Subject类的强转，我们就可以得到一个SubjectProxy，也就是xxxProxy类的实例。而且这是通用的。

只要我们依次提供给Proxy三个参数：由什么classloader加载类，动态代理这个类继承自什么接口(接口里有什么方法)，以及如何处理代理真实类执行方法(如何包装)。就能创建动态代理实例。


有一篇文章，更是详细分析了动态代理，有空可去细看。

<a>https://www.ibm.com/developerworks/cn/java/j-lo-proxy1/index.html</a>


### 加一层封装

按照以上做法，动态代理确实成功了。但是传入的参数非常多，如果我们把它们加一层封装就非常棒了。我们可以在DynamicProxy，也就是动态调用器中加一个方法来封装，解决我们的需求。

``` java

public class DynamicProxy implements InvocationHandler{
  private Object realSubject;

  public DynamicProxy(Object realSubject){
    this.realSubject = realSubject;
  }

  @SuppressWarnings("unchecked")
  public <T> T getProxy(){
    return (T) Proxy.newProxyInstance(
    realSubject.getClass().getClassLoader(),
    realSubject.getClass().getInterfaces(),
    this
    );
  }
...

}
```
如此一来，我们直接把创建动态实例的方法封装到了调用器里。而且这里还是返回的一个泛型。这样我们就可以如下调用：

``` java
DynamicProxy dynamicProxy = new DynamicProxy(new RealSubject());

Subject subjectProxy = dynamicProxy.getProxy();

subjectProxy.action();
```

如上调用就简单多了。我们注意定义getProxy方法的时候，利用了泛型，因为newProxyInstance方法返回的是一个Object对象，需要转型，所以这里安排了一个泛型参数T，在调用getProxy方法的时候，具体类型Subject大概会成为这个参数T，这样就解决了需要指定强转类型的问题。
