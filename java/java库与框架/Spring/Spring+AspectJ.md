## Spring+AspectJ
Spring集成了 AspectJ，同时也保留了以上提到的切面与代理配置方式（为了兼容老的项目，更为了维护自己的面子）。将 Spring 与 AspectJ 集成与直接使用 AspectJ 是不同的，我们不需要定义 AspectJ 类（它是扩展了 Java 语法的一种新的语言，还需要特定的编译器），只需要使用 AspectJ 切点表达式即可（它是比正则表达式更加友好的表现形式）。


### Spring + AspectJ（基于注解：通过 AspectJ execution 表达式拦截方法）

下面以一个最简单的例子，实现之前提到的环绕增强。先定义一个 Aspect 切面类：

``` java
@Aspect
@Component
public class GreetingAspect {

    @Around("execution(* aop.demo.GreetingImpl.*(..))")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        before();
        Object result = pjp.proceed();
        after();
        return result;
    }

    private void before() {
        System.out.println("Before");
    }

    private void after() {
        System.out.println("After");
    }
}
```

注意：类上面标注的 \@Aspect 注解，这表明该类是一个 Aspect（其实就是 Advisor）。该类无需实现任何的接口，只需定义一个方法（方法叫什么名字都无所谓），只需在方法上标注 \@Around 注解，在注解中使用了 AspectJ 切点表达式。方法的参数中包括一个 ProceedingJoinPoint 对象，它在 AOP 中称为 Joinpoint（连接点），可以通过该对象获取方法的任何信息，例如：方法名、参数等。

下面重点来分析一下这个切点表达式：execution(* aop.demo.GreetingImpl.*(..))

 - execution()：表示拦截方法，括号中可定义需要匹配的规则。

 - 第一个“*”：表示方法的返回值是任意的。

 - 第二个“*”：表示匹配该类中所有的方法。

 - (..)：表示方法的参数是任意的。


如果想匹配指定的方法，只需将第二个“*”改为指定的方法名称即可。

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="aop.demo"/>

    <aop:aspectj-autoproxy proxy-target-class="true"/>

</beans>
```

两行配置就行了，不需要配置大量的代理，更不需要配置大量的切面，真是太棒了！需要注意的是 proxy-target-class="true" 属性，它的默认值是 false，默认只能代理接口（使用 JDK 动态代理），当为 true 时，才能代理目标类（使用 CGLib 动态代理）。

Spring 与 AspectJ 结合的威力远远不止这些,还可以拦截指定注解。


### Spring + AspectJ（基于注解：通过 AspectJ @annotation 表达式拦截方法）

为了拦截指定的注解的方法，我们首先需要来自定义一个注解：

``` java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Tag {
}
```

以上定义了一个 \@Tag 注解，此注解可标注在方法上，在运行时生效。

只需将前面的 Aspect 类的切点表达式稍作改动：

``` java
@Aspect
@Component
public class GreetingAspect {

    @Around("@annotation(aop.demo.Tag)")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        ...
    }

    ...
}
```

这次使用了 **\@annotation() 表达式**，只需在括号内定义需要拦截的注解名称即可。

直接将 \@Tag 注解定义在您想要拦截的方法上，就这么简单：

``` java
@Component
public class GreetingImpl implements Greeting {

    @Tag
    @Override
    public void sayHello(String name) {
        System.out.println("Hello! " + name);
    }
}
```

以上示例中只有一个方法，如果有多个方法，我们只想拦截其中某些时，这种解决方案会更加有价值。

除了 \@Around 注解外，其实还有几个相关的注解，稍微归纳一下吧：

 - \@Before：前置增强

 - \@After：后置增强

 - \@Around：环绕增强

 - \@AfterThrowing：抛出增强

 - \@DeclareParents：引入增强

此外还有一个 \@AfterReturning（返回后增强），也可理解为 Finally 增强，相当于 finally 语句，它是在方法结束后执行的，也就说说，它比 \@After 还要晚一些。

### Spring + AspectJ（引入增强）

为了实现基于 AspectJ 的引入增强，我们同样需要定义一个 Aspect 类：

``` java
@Aspect
@Component
public class GreetingAspect {

    @DeclareParents(value = "aop.demo.GreetingImpl", defaultImpl = ApologyImpl.class)
    private Apology apology;
}
```


只需要在 Aspect 类中定义一个需要引入增强的接口，它也就是运行时需要动态实现的接口。在这个接口上标注了 \@DeclareParents 注解，该注解有两个属性：

 - value：目标类

 - defaultImpl：引入接口的默认实现类

我们只需要对引入的接口提供一个默认实现类即可完成引入增强：

``` java
public class ApologyImpl implements Apology {

    @Override
    public void saySorry(String name) {
        System.out.println("Sorry! " + name);
    }
}
```

以上这个实现会在运行时自动增强到 GreetingImpl 类中，也就是说，无需修改 GreetingImpl 类的代码让它去实现 Apology 接口，我们单独为该接口提供一个实现类（ApologyImpl），来做 GreetingImpl 想做的事情。


客户端的使用

``` java

public class Client {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("aop/demo/spring.xml");
        Greeting greeting = (Greeting) context.getBean("greetingImpl");
        greeting.sayHello("Jack");

        Apology apology = (Apology) greeting; // 强制转型为 Apology 接口
        apology.saySorry("Jack");
    }
}
```

从 Spring ApplicationContext 中获取 greetingImpl 对象（其实是个代理对象），可转型为自己静态实现的接口 Greeting，也可转型为自己动态实现的接口 Apology，切换起来非常方便。

使用 AspectJ 的引入增强比原来的 Spring AOP 的引入增强更加方便了，而且还可面向接口编程（以前只能面向实现类），这也算一个非常巨大的突破。

这一切真的已经非常强大也非常灵活了！但仍然还是有用户不能尝试这些特性，因为他们还在使用 JDK 1.4（根本就没有注解这个东西），怎么办呢？没想到 Spring AOP 为那些遗留系统也考虑到了。

###  Spring + AspectJ（基于配置）

除了使用 \@Aspect 注解来定义切面类以外，Spring AOP 也提供了基于配置的方式来定义切面类：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...>

    <bean id="greetingImpl" class="aop.demo.GreetingImpl"/>

    <bean id="greetingAspect" class="aop.demo.GreetingAspect"/>

    <aop:config>
        <aop:aspect ref="greetingAspect">
            <aop:around method="around" pointcut="execution(* aop.demo.GreetingImpl.*(..))"/>
        </aop:aspect>
    </aop:config>

</beans>
```

使用 \<aop:config\> 元素来进行 AOP 配置，在其子元素中配置切面，包括增强类型、目标方法、切点等信息。

无论您是不能使用注解，还是不愿意使用注解，Spring AOP 都能为您提供全方位的服务。

以上就是基础的Spring AOP技术了。
