## proxy包

处理动态代理功能的package，依赖CGLib库，有关CGLib可见java库的笔记。

### Proxy接口

Proxy接口中包括了一个doProxy方法，传入一个ProxyChain，用于执行“链式代理”操作。

链式代理，就是可将多个代理通过一个链子串起来，一个个地执行，按照先后顺序。

实现中会提供相应的横切逻辑(也就是增强Advice的逻辑)，并且再次调用proxyChain的doProxyChain()方法。

这里一开始我有个困惑，那就是proxy方法如何控制前置和后置的增强。实质上上proxy类中的doProxy类，就是通过调用doProxyChain的时机来决定前置增强还是后置增强的。因为一旦调用doProxyChain，代理链就会逐条执行，直到最后调用methodProxy(拦截器)来执行原方法，但是执行完原方法后还是会回到(return到)这个doProxy方法来

```
.....//前置增强逻辑
result = proxyChain.doProxyChain();
.....//后置增强逻辑
```

如上所示，doProxy里，调用完链式方法还是会返回值的。得到这个返回值后就可以进行后置逻辑的处理。最后再把这个result return回去就可以了。


### ProxyChain类
类中定义了一系列成员变量，核心方法是doProxyChain方法，用来一一执行proxy类.类里有proxyIndex变量，记录内置proxyList中proxy的数量。执行doProxyChain时，从0开始执行list中proxy类的doProxy方法，而这个方法中也会再次调用proxyChain中的doProxyChain方法，所以相当于proxyList中的代理会被一一执行完毕。执行完毕以后，调用MethodProxy中的invokeSuper方法，执行realObject的业务逻辑。

### ProxyManager类
这个类便是用来创建代理的了，我们传入realObject类的Class类，以及一些proxy接口的实现类的List，它便能返回给我们一个**xxxProxy代理类的实例**。

原理便是使用CGLib，new一个匿名MethodInterceptor对象，并覆盖
里面的intercept拦截方法，原本在这个方法里便可以实现增强拦截等效果。这时把MethodProxy这个方法拦截器对象传出去，传给proxyChain对象，就可以依次执行完串起来的proxy方法(proxyChain的逻辑)，最后再调用MethodProxy.invokeSuper来继续realObject的逻辑。

用Enhancer.create方法来创建并返回代理。

### AspectProxy抽象类
继承了Proxy接口，相当于Proxy实现的一个**模板抽象类**，所有的proxy类都要继承这个切面类。这个切面类里封装了before，after等方法，用来指导如何增强从targetObject拦截来的方法。还有intercept用来做拦截条件判断。

从代码可知，在将proxychain执行前，会执行before方法，proxychain执行并得到最终结果后，还会调用after方法实现后置增强。这个方法执行时会从传入的proxychain中拿出执行的方法，对象，参数等。除此之外，抽象方法还能有选择的实现。

之后用户想创建proxy，直接继承这个模板就方便很多了。
