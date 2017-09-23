## Spring框架的AOP
Spring 框架的一个关键组件是面向方面的编程(AOP)框架。

面向方面的编程需要把程序逻辑分解成不同的部分称为所谓的关注点。这些横切关注点在概念上独立于应用程序的业务逻辑。有各种各样的常见的很好的方面的例子，如**日志记录、审计、声明式事务、安全性和缓存**等。

拿性能监控举例，假如我们想测试方法的执行时间，最普通的做法就是在这个方法执行前后获取系统时间，然后相减。但是这样的效率实在是太低了，每个方法都需要你去改代码。但是根据AOP的思想，我们就可以以方法为"切点"，用代理模式的思想去给方法进行包装，然后代理执行(可见代理模式笔记)。也就是用代理模式包装之后，就可以在指定方法调用前后执行before()和after().这相当于把方法进行了切割，所以这便是AOP思想。


### AOP术语

<table>
<thead>
<tr>
<th>术语</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td>方面/切面(<code>Aspect</code>)</td>
<td>一个具有一组API的模块，提供交叉要求。例如，日志记录模块被称为AOP方面用于记录。应用程序可以根据需要具有任意数量的方面。</td>
</tr>
<tr>
<td>加入点(<code>Join point</code>)</td>
<td>这表示您的应用程序中可以插入AOP方面的一点。也可以说，这是应用程序中使用Spring AOP框架采取操作的实际位置。</td>
</tr>
<tr>
<td>通知(<code>Advice</code>)</td>
<td>这是在方法执行之前或之后采取的实际操作。 这是在Spring AOP框架的程序执行期间调用的实际代码片段。</td>
</tr>
<tr>
<td>切入点(<code>Pointcut</code>)</td>
<td>这是一组一个或多个连接点，其中应该执行通知(<code>Advice</code>)。 您可以使用表达式或模式指定切入点，我们将在AOP示例中看到。</td>
</tr>
<tr>
<td>介绍(<code>Introduction</code>)</td>
<td>介绍允许向现有类添加新的方法或属性。</td>
</tr>
<tr>
<td>目标对象(<code>Target object</code>)</td>
<td>对象被一个或多个方面通知(<code>Advice</code>)，该对象将始终是代理的对象。也称为通知(<code>Advice</code>)对象。</td>
</tr>
<tr>
<td>编织(<code>Weaving</code>)</td>
<td>编织是将方面与其他应用程序类型或对象进行链接以创建通知(<code>Advice</code>)对象的过程。 这可以在编译时，加载时间或运行时完成。</td>
</tr>
</tbody>
</table>

上面的术语可以在之后慢慢理解。

### 前置，后置，环绕增强(编程式)

实质上，所谓的Before Advice(前置增强),After Advice(后置增强)，也就是相当于代理模式中的，在被代理的方法执行前后所执行的方法(可见代理模式笔记)。

before方法和after方法合并在一块的话，就是环绕增强(around
 advice).

前后置增强的实现

``` java

public class GreetingBeforeAdvice implements MethodBeforeAdvice {

  @Override
  public void before(Method method,Object[] args, Object target) throws Throwable {
    System.out.println("Before");
  }

}


public class GreetingAfterAdvice implements AfterReturningAdvice {

  @Override
  public void afterReturning(Object result,Method method,Object[] args,Object target) throws Throwable {
    System.out.println("After");
  }

}



```


前置增强实现了org.springframework.aop.MethodBeforeAdvice接口，后置增强则实现了org.springframework.aop.AfterReturningAdvice接口。

最后我们可以客户端集成

``` java

public class Client{

  public static void main(String args[]){
    //创建代理工厂
    ProxyFactory.proxyFactory = new ProxyFactory();

    //射入目标对象realObject
    proxyFactory.setTarget(new GreetingImpl());

    //添加前置增强
    proxyFactory.addAdvice(new GreetingBeforeAdvice());

    //添加后置增强
    proxyFactory.addAdvice(new GreetingAfterAdvice());

    //调用方法获取代理
    Greeting greeting = (Greeting) proxyFactory.getProxy();

    //调用xxxProxy类的方法
    greeting.action();
  }


}

```


当然我们也可以定义一个类同时实现两个接口

``` java

public class GreetingBeforeAndAfterAdvice
implements MethodBeforeAdvice,AfterReturningAdvice
{...}

//然后调用
proxyFactory.addAdvice(new GreetingBeforeAndAfterAdvice());

```

像上面那样调用也是可以的。

之前还有说到环绕增强，实现环绕增强则需要实现org.aopalliance.intercept.MethodInterceptor接口。这个接口并不是Spring提供的，而是AOP联盟写的，Spring只是借用。实现如下

``` java
public class GreetingAroundAdvice implements MethodInterceptor {
  @Override
  public Object invoke(MethodInvocation invocation) throws Throwable{
    before();
    Object result = invocation.proceed();
    after();
    return result;
  }


  private void before(){...}
  private void after(){...}
}
```

客户端中，调用addAdvice，把刚才的环绕增强类放入即可，跟前面一样。

### 前后环绕增强(声明式)
刚才使用了编程式的方法实现了AOP，但是这仍然很不方便，如果可以用配置文件的方式来定义Bean对象，把new语句都解放掉，就好多了。

以下是Spring的配置文件

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 扫描指定包（将 @Component 注解的类自动定义为 Spring Bean） -->
    <context:component-scan base-package="aop.demo"/>

    <!-- 配置一个代理 -->
    <bean id="greetingProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="interfaces" value="aop.Greeting"/> <!-- 需要代理的接口 -->
        <property name="target" ref="greetingImpl"/>       <!-- 接口实现类 -->
        <property name="interceptorNames">                 <!-- 拦截器名称（也就是增强类名称，Spring Bean 的 id） -->
            <list>
                <value>greetingAroundAdvice</value>
            </list>
        </property>
    </bean>

</beans>
```

仔细观察其实很好理解，interceptorName其实就是需要addAdvice的内容。需要注意一点就是，配置文件中可以加入一条设置

``` xml
<property name="proxyTargetClass" value="true"></property>
```

这个设置表明是否代理目标类，默认为false，也就是代理接口，此时Spring默认使用JDK动态代理，如果为true，用的是CGLib代理，一切都封装好了，非常方便。



客户端则可以如此写

``` java

public class Client {

  public static void main(String[] args){
    // 获取xml文件
    ApplicationContext context = new ClassPathXmlApplicationContext("aop/demo/spring.xml");

    //获取bean对象
    Greeting greeting = (Greeting)context.getBean("greetingProxy");

    greeting.action();
  }

}

```

这样一来，确实方便了很多。


### 抛出增强
其实除了前置增强，后置增强，环绕增强意外，还有其他类别的增强方式，比如**抛出增强**。

比如程序报错，我们需要调试，一般做法是打印信息到控制台或者日志文件。但是这样一来我们很多地方都需要做处理，如果使用Throws Advice(抛出增强),就可以一劳永逸了。

抛出增强类需要实现org.springframework.aop.ThrowsAdvice接口，在接口方法中可以获取方法，参数，目标对象，异常对象等信息。

``` java

@Component
public class GreetingThrowAdvice implements ThrowsAdvice {
  public void afterThrowing(Method method,Object[] args,Object target,Exception e){
    System.out.println("**Throw Exception**");
    System.out.println("Target Class:" + target.getClass().getName());
    System.out.println("Method Name:" + method.getName());
    System.out.println("Exception Message: "  e.getMessage());
    System.out.println("************");

  }
}

```

然后我们便可以把这些都写到日志当中，也可以持久化到数据库。


### 引入增强
还有一个更强的增强方式，叫**引入增强**(Introduction Advice).

这是对类的增强。

假如我们又定义了一个接口

``` java
public interface Apology{
  void saySorry(String name);
}

```

我们想让实现greeting接口的realObject，greetingImpl类来执行saySorry方法，然而这样的话，我们则需要让这个类在代码中实现Apology接口，这需要我们修改代码，真实情景可能非常复杂。这时，引入增强就能帮我们解决这个问题。

``` java

@Component
public class GreetingIntroAdvice extends DelegatingIntroductionInterceptor implements Apology {

  @Override
  public Object invoke(MethodInvocation invocation) throws Throwable {
    return super.invoke(invocation);
  }

  @Override
  public void saySorry(String name){
    System.out.println("Sorry! " + name);
  }
}


```

以上定义了一个引入增强类，扩展了org.springframework.aop.support.DelegatingIntroductionInterceptor类，同时也实现了新定义的Apology接口。覆写了invoke方法，还实现了Apology接口。然后我们无需直接实现Apology接口，就可以在程序运行时直接调用方法，非常神奇！

同样，利用配置文件，就能添加入这种增强，具体配置文件就省去了。客户端代码如下


``` java

public class Client {

  public static void main(String[] args){
    // 获取xml文件
    ApplicationContext context = new ClassPathXmlApplicationContext("aop/demo/spring.xml");

    //获取bean对象,注意，配置文件proxyTargetClass设置为true，可以直接代理类而不是接口
    GreetingImpl greeting = (GreetingImpl)context.getBean("greetingProxy");
    greeting.action()；

    //这里允许直接强转成Apology接口，相当强大。
    Apology apology = (Apology) greetingImpl;

  }

}

```
