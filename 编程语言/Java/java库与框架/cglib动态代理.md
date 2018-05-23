## CGLib动态代理
jdk动态代理很好，但是也有一个缺陷，就是如果想代理一个没有实现任何接口的类，它就没有用武之处了。所以还有一种动态代理的方式，那便是CGLibProxy

 Cglib是一个优秀的动态代理框架，**它的底层使用ASM（JAVA字节码处理框架）在内存中动态的生成被代理类的子类**。使用CGLIB即使被代理类没有实现任何接口也可以实现动态代理功能。但是不能对final修饰的类进行代理。

 ### 使用流程

首先定义一个没有实现接口的代理委托类(realObject类)CglibRealSubject.java.

``` java
/**
 * 没有实现接口的代理委托类
 *
 */
public class CglibRealSubject{

    public void visit() {
        System.out.println("I am 'RealSubject',I am the execution method");
    }

}
```

接着定义一个Cglib动态代理类: CglibProxy.java

``` java
public class CGLibProxy implements MethodInterceptor{

  public <T> T getProxy(Class<T> cls){
    return (T) Enhancer.create(cls,this);
  }

  public Object intercept(Object obj,Method method,Object[] args,MethodProxy proxy) throws Throwable {
    before();
    Object result = proxy.invoke(obj,args);
    after();

    return result;

  }



}



```

我们需要实现CGLib给我们提供的MethodInterceptor实现类，并填充intercept方法，方法中最后有个MethodProxy参数，这个就是传说中的方法拦截器，invokeSuper，将被代理的对象以及args传入即可。

我们也在这个类里添加了getProxy方法，返回泛型，帮助我们直接构建xxxProxy类。

main中调用时

``` java
public static void main(String[] args){
  CGLibProxy cgLibProxy = new CGLibProxy();

  RealObject realObjectProxy = cgLibProxy.getProxy(RealObject.class);

  realObjectProxy.visit();
}
```


简而言之，通过CGLib，可以解决jdk动态代理无法代理没有实现接口的realObject的问题。
