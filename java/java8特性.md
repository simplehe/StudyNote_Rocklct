## java8新特性
Java 8 (又称为 jdk 1.8) 是 Java 语言开发的一个主要版本。 Oracle 公司于 2014 年 3 月 18 日发布 Java 8 ，它支持函数式编程，新的 JavaScript 引擎，新的日期 API，新的Stream API 等。

### 函数式接口
Java 8 引入的一个核心概念是函数式接口（Functional Interfaces）。通过在接口里面添加一个抽象方法，这些方法可以直接从接口中运行。**如果一个接口定义个唯一一个抽象方法，那么这个接口就成为函数式接口**。

同时，引入了一个新的注解：@FunctionalInterface。可以把他它放在一个接口前，表示这个接口是一个函数式接口。这个注解是非必须的，**只要接口只包含一个方法的接口**，虚拟机会自动判断，不过最好在接口上使用注解 \@FunctionalInterface 进行声明。**在接口中添加了 \@FunctionalInterface 的接口，只允许有一个抽象方法，否则编译器也会报错**。


java.lang.Runnable 就是一个函数式接口。

``` java
@FunctionalInterface
public interface Runnable {
public abstract void run();
}
```

### Lambda 表达式
函数式接口的重要属性是：我们能够使用 Lambda 实例化它们，Lambda 表达式让你能够将函数作为方法参数，或者将代码作为数据对待。Lambda 表达式的引入给开发者带来了不少优点：在 Java 8 之前，匿名内部类，监听器和事件处理器的使用都显得很冗长，代码可读性很差，**Lambda 表达式的应用则使代码变得更加紧凑，可读性增强**；Lambda 表达式使并行操作大集合变得很方便，可以充分发挥多核 CPU 的优势，更易于为多核处理器编写代码；

Lambda 表达式由三个部分组成：第一部分为一个括号内用逗号分隔的形式参数，参数是函数式接口里面方法的参数；第二部分为一个箭头符号：->；第三部分为方法体，可以是表达式和代码块。语法如下：

1. 方法体为表达式，该表达式的值作为返回值返回。

``` java
(parameters) -> expression
```

2. 方法体为代码块，必须用 {} 来包裹起来，且需要一个 return 返回值，但若函数式接口里面方法返回值是 void，则无需返回值。

``` java
(parameters) -> { statements; }
```

简单对比

``` java
button.addActionListener(new ActionListener() {
@Override
public void actionPerformed(ActionEvent e) {
System.out.print("Helllo Lambda in actionPerformed");
}
});
下面是使用 Lambda 表达式后：
button.addActionListener(
\\actionPerformed 有一个参数 e 传入，所以用 (ActionEvent e)
(ActionEvent e)->
System.out.print("Helllo Lambda in actionPerformed")
);
```

要使用 Lambda 表达式，需要定义一个函数式接口，这样往往会让程序充斥着过量的仅为 Lambda 表达式服务的函数式接口。为了减少这样过量的函数式接口，Java 8 在 java.util.function 中增加了不少新的函数式通用接口。例如：

Function\<T, R\>：将 T 作为输入，返回 R 作为输出，他还包含了和其他函数组合的默认方法。

Predicate\<T\> ：将 T 作为输入，返回一个布尔值作为输出，该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（与、或、非）。

Consumer\<T\> ：将 T 作为输入，不返回任何内容，表示在单个参数上的操作。

### 增强接口
Java 8 对接口做了**进一步的增强**。在接口中**可以添加使用 default 关键字修饰的非抽象方法**。还可以在接口中**定义静态方法**。如今，接口看上去与抽象类的功能越来越类似了。

Java 8 还允许我们给接口添加一个非抽象的方法实现，只需要使用 default 关键字即可，这个特征又叫做扩展方法。在实现该接口时，该默认扩展方法在子类上可以直接使用，它的使用方式类似于抽象类中非抽象成员方法。但扩展方法不能够重载 Object 中的方法。例如：toString、equals、 hashCode 不能在接口中被重载。

### 集合的流式操作
Java 8 引入了流式操作（Stream），通过该操作可以实现对集合（Collection）的并行处理和函数式操作。根据操作返回的结果不同，流式操作分为中间操作和最终操作两种。最终操作返回一特定类型的结果，而中间操作返回流本身，这样就可以将多个操作依次串联起来。根据流的并发性，流又可以分为串行和并行两种。流式操作实现了集合的过滤、排序、映射等功能。

Stream 和 Collection 集合的区别：Collection 是一种静态的内存数据结构，而 Stream 是有关计算的。前者是主要面向内存，存储在内存中，后者主要是面向 CPU，通过 CPU 实现计算。

流有串行和并行两种，串行流上的操作是在一个线程中依次完成，而**并行流则是在多个线程上同时执行**。并行与串行的流可以相互切换：通过 stream.sequential() 返回串行的流，通过 stream.parallel() 返回并行的流。相比较串行的流，并行的流可以很大程度上提高程序的执行效率。


并行排序：

``` java
List<String> list = new ArrayList<String>();
for(int i=0;i<1000000;i++){
double d = Math.random()*1000;
list.add(d+"");
}
long start = System.nanoTime();//获取系统开始排序的时间点
int count = (int)((Stream) list.stream().parallel()).sorted().count();
long end = System.nanoTime();//获取系统结束排序的时间点
long ms = TimeUnit.NANOSECONDS.toMillis(end-start);//得到并行排序所用的时间
System.out.println(ms+”ms”);
//串行输出为 1200ms，并行输出为 800ms。可见，并行排序的时间相比较串行排序时间要少不少。
```

#### 中间操作
该操作会保持 stream 处于中间状态，允许做进一步的操作。它返回的还是的 Stream，允许更多的链式操作。常见的中间操作有：

```
filter()：对元素进行过滤；
sorted()：对元素排序；
map()：元素的映射；
distinct()：去除重复元素；
subStream()：获取子 Stream 等。
```

``` java
list.stream()
.filter((s) -> s.startsWith("s"))
.forEach(System.out::println);
```

#### 终止操作
该操作必须是流的最后一个操作，一旦被调用，Stream 就到了一个终止状态，而且不能再使用了。常见的终止操作有：

```
forEach()：对每个元素做处理；
toArray()：把元素导出到数组；
findFirst()：返回第一个匹配的元素；
anyMatch()：是否有匹配的元素等。
```

### 注解的更新
对于注解，Java 8 主要有两点改进：类型注解和重复注解。

Java 8 的类型注解扩展了注解使用的范围。在该版本之前，注解只能是在声明的地方使用。现在**几乎可以为任何东西添加注解**：**局部变量、类与接口，就连方法的异常也能添加注解**。新增的两个注释的程序元素类型 ElementType.TYPE_USE 和 ElementType.TYPE_PARAMETER 用来描述注解的新场合。ElementType.TYPE_PARAMETER 表示该注解能写在类型变量的声明语句中。而 ElementType.TYPE_USE 表示该注解能写在使用类型的任何语句中（例如声明语句、泛型和强制转换语句中的类型）。

对类型注解的支持，增强了通过静态分析工具发现错误的能力。原先只能在运行时发现的问题可以提前在编译的时候被排查出来。Java 8 本身虽然没有自带类型检测的框架，但可以通过使用 Checker Framework 这样的第三方工具，自动检查和确认软件的缺陷，提高生产效率。

### IO/NIO 的改进
Java 8 对 IO/NIO 也做了一些改进。主要包括：改进了 java.nio.charset.Charset 的实现，使编码和解码的效率得以提升，也精简了 jre/lib/charsets.jar 包；优化了 String(byte[],*) 构造方法和 String.getBytes() 方法的性能；还增加了一些新的 IO/NIO 方法，使用这些方法可以从文件或者输入流中获取流（java.util.stream.Stream），通过对流的操作，可以简化文本行处理、目录遍历和文件查找。
新增的 API 如下：

BufferedReader.line(): 返回文本行的流 Stream\<String\>

File.lines(Path, Charset):返回文本行的流 Stream\<String\>

File.list(Path): 遍历当前目录下的文件和目录

File.walk(Path, int, FileVisitOption): 遍历某一个目录下的所有文件和指定深度的子目录

File.find(Path, int, BiPredicate, FileVisitOption... ): 查找相应的文件
