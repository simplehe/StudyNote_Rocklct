## java注解
注解是 **Java 5** 的一个新特性。注解是插入你代码中的一种注释或者说是一种元数据（meta data）。**这些注解信息可以在编译期使用预编译工具进行处理（pre-compiler tools），也可以在运行期使用 Java 反射机制进行处理。**

有很多非常有名的框架，比如Hibernate、Spring等框架中都大量使用注解。

### 注解定义

``` java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface FruitColor {

    enum Color{RED,YELLOW,WHITE}

    Color fruitColor() default Color.RED;

}
```

在interface前面的@符号表名这是一个注解，一旦你定义了一个注解之后你就可以将其应用到你的代码中

### 注解使用
注解可以用来修饰代码中的这些元素

1. 类
2. 接口
3. 方法
4. 方法参数
5. 属性
6. 局部变量

以下为使用
``` java
@MyAnnotation(name="someName",  value = "Hello World")
public class TheClass {
}

```

### 元注解
元注解的作用就是负责注解其他注解。**Java5.0定义了4个标准的meta-annotation类型**，它们被用来提供对其它 annotation类型作说明。Java5.0定义的元注解：

1. \@Target,
2. \@Retention,
3. \@Documented,
4. \@Inherited


**这些类型和它们所支持的类在java.lang.annotation包中可以找到**。也就是说各种ElementType之类的枚举值等都在这个包中。



#### \@Target
\@Target说明了Annotation所修饰的对象范围：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。

**作用：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）**

取值(ElementType)有：

1. CONSTRUCTOR:用于描述构造器
2. FIELD:用于描述域即类成员变量
3. LOCAL_VARIABLE:用于描述局部变量
4. METHOD:用于描述方法
5. PACKAGE:用于描述包
6. PARAMETER:用于描述参数
7. TYPE:用于描述类、接口(包括注解类型) 或enum声明

``` java
@Target(ElementType.TYPE)
public @interface Table {
    /**
     * 数据表名称注解，默认值为类名称
     * @return
     */
    public String tableName() default "className";
}

@Target(ElementType.FIELD)
public @interface NoDBColumn {

}
```

注解Table 可以用于注解类、接口(包括注解类型)或enum声明,而注解NoDBColumn仅可用于注解类的成员变量。

#### \@Retention
\@Retention定义了该Annotation被保留的时间长短：某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对 Annotation的“生命周期”限制。

**作用：表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）**

取值（RetentionPoicy）有：

1. SOURCE:在源文件中有效（即源文件保留）
2. CLASS:在class文件中有效（即class保留）
3. RUNTIME:在运行时有效（即运行时保留）

``` java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Column {
    public String name() default "fieldName";
    public String setFuncName() default "setField";
    public String getFuncName() default "getField";
    public boolean defaultDBValue() default false;
}
```

Column注解的的RetentionPolicy的属性值是RUTIME,这样注解处理器可以通过反射，获取到该注解的属性值，从而去做一些运行时的逻辑处理。

#### \@Documented:

\@Documented用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员。

``` java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Column {
    public String name() default "fieldName";
    public String setFuncName() default "setField";
    public String getFuncName() default "getField";
    public boolean defaultDBValue() default false;
}
```

#### \@Inherited

\@Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。**如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类**。

　　注意：@Inherited annotation类型是被标注过的class的子类所继承。类并不从它所实现的接口继承annotation，方法并不从它所重载的方法继承annotation。

　　当@Inherited annotation类型标注的annotation的Retention是RetentionPolicy.RUNTIME，则反射API增强了这种继承性。如果我们使用java.lang.reflect去查询一个@Inherited annotation类型的annotation时，反射代码检查将展开工作：检查class和其父类，直到发现指定的annotation类型被发现，或者到达类继承结构的顶层.

``` java
@Inherited
public @interface Greeting {
    public enum FontColor{ BULE,RED,GREEN};
    String name();
    FontColor fontColor() default FontColor.GREEN;
}
```
