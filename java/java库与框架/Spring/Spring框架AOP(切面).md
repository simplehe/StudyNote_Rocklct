## Spring AOP 切面
在增强方式里提到的拦截器，都只是拦截整个类，如果只想拦截特定的方法，还要判断方法名然后过滤。这种方式非常不优雅.

而AOP中一个很重要的概念**切面--Advisor**,则能帮助我们解决这一个问题。切面可以帮助我们拦截匹配条件。这个“拦截匹配条件”在AOP中，就叫做**切点**。说白了就是一个基于表达式的拦截条件。

**Advisor(切面)封装了Advice(增强)和Pointcut(切点)**


在 GreetingImpl 类中故意增加了两个方法，都以“good”开头。下面要做的就是拦截这两个新增的方法，而对 sayHello() 方法不作拦截。

``` java
@Component
public class GreetingImpl implements Greeting {

    @Override
    public void sayHello(String name) {
        System.out.println("Hello! " + name);
    }

    public void goodMorning(String name) {
        System.out.println("Good Morning! " + name);
    }

    public void goodNight(String name) {
        System.out.println("Good Night! " + name);
    }
}
```

在 Spring AOP 中，已经给我们提供了许多切面类了，以正则表达式切面类举例：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...>

    <context:component-scan base-package="aop.demo"/>

    <!-- 配置一个切面 -->
    <bean id="greetingAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
        <property name="advice" ref="greetingAroundAdvice"/>            <!-- 增强 -->
        <property name="pattern" value="aop.demo.GreetingImpl.good.*"/> <!-- 切点（正则表达式） -->
    </bean>

    <!-- 配置一个代理 -->
    <bean id="greetingProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="greetingImpl"/>                <!-- 目标类 -->
        <property name="interceptorNames" value="greetingAdvisor"/> <!-- 切面 -->
        <property name="proxyTargetClass" value="true"/>            <!-- 代理目标类 -->
    </bean>

</beans>
```

注意以上代理对象的配置中的 interceptorNames，它不再是一个增强，而是一个切面，因为已经将增强封装到该切面中了。此外，切面还定义了一个切点（正则表达式），其目的是为了只将满足切点匹配条件的方法进行拦截。

需要强调的是，这里的切点表达式是基于正则表达式的。示例中的“aop.demo.GreetingImpl.good.\*”表达式后面的“.\*”表示匹配所有字符，翻译过来就是“匹配 aop.demo.GreetingImpl 类中以 good 开头的方法”。


除了 RegexpMethodPointcutAdvisor 以外，在 Spring AOP 中还提供了几个切面类，比如：

 - DefaultPointcutAdvisor：默认切面（可扩展它来自定义切面）

 - NameMatchMethodPointcutAdvisor：根据方法名称进行匹配的切面

 - StaticMethodMatcherPointcutAdvisor：用于匹配静态方法的切面

总的来说，让用户去配置一个或少数几个代理，似乎还可以接受，但随着项目的扩大，代理配置就会越来越多，配置的重复劳动就多了，麻烦不说，还很容易出错。能否让 Spring 框架为我们自动生成代理呢？


### 自动代理(扫描Bean名称)

Spring AOP 提供了一个可根据 Bean 名称来自动生成代理的工具，它就是 BeanNameAutoProxyCreator。是这样配置的：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...>

    ...

    <bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
        <property name="beanNames" value="*Impl"/>                       <!-- 只为后缀是“Impl”的 Bean 生成代理 -->
        <property name="interceptorNames" value="greetingAroundAdvice"/> <!-- 增强 -->
        <property name="optimize" value="true"/>                         <!-- 是否对代理生成策略进行优化 -->
    </bean>

</beans>
```

以上使用 BeanNameAutoProxyCreator 只为后缀为“Impl”的 Bean 生成代理。需要注意的是，这个地方我们不能定义代理接口，也就是 interfaces 属性，因为我们根本就不知道这些 Bean 到底实现了多少接口。此时不能代理接口，而只能代理类。所以这里提供了一个新的配置项，它就是 optimize。若为 true 时，则可对代理生成策略进行优化（默认是 false 的）。也就是说，如果该类有接口，就代理接口（使用 JDK 动态代理）；如果没有接口，就代理类（使用 CGLib 动态代理）。而并非像之前使用的 proxyTargetClass 属性那样，强制代理类，而不考虑代理接口的方式。可见 Spring AOP 确实为我们提供了很多很好地服务！


既然 CGLib 可以代理任何的类了，那为什么还要用 JDK 的动态代理呢？肯定您会这样问。

根据多年来实际项目经验得知：CGLib 创建代理的速度比较慢，但创建代理后运行的速度却非常快，而 JDK 动态代理正好相反。如果在运行的时候不断地用 CGLib 去创建代理，系统的性能会大打折扣，**所以建议一般在系统初始化的时候用 CGLib 去创建代理，并放入 Spring 的 ApplicationContext 中以备后用**。


### 自动代理(扫描切面配置)

为了匹配目标类中的指定方法，我们仍然需要在 Spring 中配置切面与切点：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...>

    ...

    <bean id="greetingAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
        <property name="pattern" value="aop.demo.GreetingImpl.good.*"/>
        <property name="advice" ref="greetingAroundAdvice"/>
    </bean>

    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator">
        <property name="optimize" value="true"/>
    </bean>

</beans>
```

这里无需再配置代理了，因为代理将会由 DefaultAdvisorAutoProxyCreator 自动生成。也就是说，这个类可以扫描所有的切面类，并为其自动生成代理。


上面两种自动生成代理的方式，一种通过扫描Bean类，一种则是通过扫描切面配置。

虽说如此，但仍然需要配置切面，略有不便。
