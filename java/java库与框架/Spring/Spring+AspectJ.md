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
