## proxy包

处理动态代理功能的package。

### Proxy接口

Proxy接口中包括了一个doProxy方法，传入一个ProxyChain，用于执行“链式代理”操作。

链式代理，就是可将多个代理通过一个链子串起来，一个个地执行，按照先后顺序。

实现中会提供相应的横切逻辑(也就是增强Advice的逻辑)，并且再次调用proxyChain的doProxyChain()方法。

### ProxyChain类
类中定义了一系列成员变量，核心方法是doProxyChain方法，用来一一执行proxy类.类里有proxyIndex变量，记录内置proxyList中proxy的数量。执行doProxyChain时，从0开始执行list中proxy类的doProxy方法，而这个方法中也会再次调用proxyChain中的doProxyChain方法，所以相当于proxyList中的代理会被一一执行完毕。执行完毕以后，调用MethodProxy中的invokeSuper方法，执行realObject的业务逻辑。

### ProxyManager类
这个类便是用来创建代理的了，我们传入realObject类，以及一些proxy接口的实现类，它便能返回给我们一个xxxProxy代理类。
