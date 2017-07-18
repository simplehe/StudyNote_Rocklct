## java异常处理
异常发生的原因有很多，通常包含以下几大类：
 - 用户输入了非法数据。
 - 要打开的文件不存在。
 - 网络通信时连接中断，或者JVM内存溢出。

要理解Java异常处理是如何工作的，你需要掌握以下三种类型的异常：
 - 检查性异常：最具代表的检查性异常是用户错误或问题引起的异常，这是程序员无法预见的。例如要打开一个不存在文件时，一个异常就发生了，这些异常在编译时不能被简单地忽略。
 - 运行时异常： 运行时异常是可能被程序员避免的异常。与检查性异常相反，运行时异常可以在编译时被忽略。
 - 错误： 错误不是异常，而是脱离程序员控制的问题。错误在代码中通常被忽略。例如，当栈溢出时，一个错误就发生了，它们在编译也检查不到的。

### Exception类
所有的异常类是从 java.lang.Exception 类继承的子类。
Exception 类是 Throwable 类的子类。除了Exception类外，Throwable还有一个子类Error 。
Java 程序通常不捕获错误。错误一般发生在严重故障时，它们在Java程序处理的范畴之外。
Error 用来指示运行时环境发生的错误。
例如，JVM 内存溢出。一般地，程序不会从错误中恢复。
异常类有两个主要的子类：IOException 类和 RuntimeException 类。

![](image/exception2.jpg)

### try-catch的规则
关于try-catch的具体用法，这里不再赘述。

首先一个需要注意的规则：除了下列情况，总将执行 finally 做为结束：**JVM 过早终止（调用 System.exit(int)）；在 finally 块中抛出一个未处理的异常；计算机断电、失火、或遭遇病毒攻击。**

先看下执行顺序
1. 当try没有捕获到异常时：try语句块中的语句逐一被执行，程序将跳过catch语句块，执行finally语句块和其后的语句；

2. 当try捕获到异常，catch语句块里没有处理此异常的情况：当try语句块里的某条语句出现异常时，而没有处理此异常的catch语句块时，此异常将会抛给JVM处理，finally语句块里的语句还是会被执行，但finally语句块后的语句不会被执行；

3. 当try捕获到异常，catch语句块里有处理此异常的情况：在try语句块中是按照顺序来执行的，当执行到某一条语句出现异常时，程序将跳到catch语句块，并与catch语句块逐一匹配，找到与之对应的处理程序，其他的catch语句块将不会被执行，而try语句块中，出现异常之后的语句也不会被执行，catch语句块执行完后，执行finally语句块里的语句，最后执行finally语句块后的语句；


### 异常抛出
throws语句用在方法定义时声明该方法要抛出的异常类型，如果抛出的是Exception异常类型，则该方法被声明为抛出所有的异常。多个异常可使用逗号分割。

throw总是出现在函数体中，用来抛出一个Throwable类型的异常。**程序会在throw语句后立即终止(重要！)**，它后面的语句执行不到.
然后在 **包含它的所有try块中(可能在上层调用函数中)** 从里向外寻找含有与其匹配的catch子句的try块。不过要注意的是，先执行完本层的finally再继续回溯找catch。

这上面一段的理解相当重要，也就是每当一个异常被throw以后，他只会被catch一次，会从里往外找可以catch的地方，一旦被catch，这个异常将会被处理完毕。当回溯到前面的方法时，异常将不会再被catch，而是直接进入finally方法(如果有的话)

本层若是无法捕捉异常，但是finally语句里有return语句，那么这个异常无法被上层捕捉，会被忽略。

看一个比较长的例子
``` java
package Test;

public class TestException {
    public TestException() {
    }

    boolean testEx() throws Exception {
        boolean ret = true;
        try {
            ret = testEx1();
        } catch (Exception e) {
            System.out.println("testEx, catch exception");
            ret = false;
            throw e;
        } finally {
            System.out.println("testEx, finally; return value=" + ret);
            return ret;
        }
    }

    boolean testEx1() throws Exception {
        boolean ret = true;
        try {
            ret = testEx2();
            if (!ret) {
                return false;
            }
            System.out.println("testEx1, at the end of try");
            return ret;
        } catch (Exception e) {
            System.out.println("testEx1, catch exception");
            ret = false;
            throw e;
        } finally {
            System.out.println("testEx1, finally; return value=" + ret);
            return ret;
        }
    }

    boolean testEx2() throws Exception {
        boolean ret = true;
        try {
            int b = 12;
            int c;
            for (int i = 2; i >= -2; i--) {
                c = b / i;
                System.out.println("i=" + i);
            }
            return true;
        } catch (Exception e) {
            System.out.println("testEx2, catch exception");
            ret = false;
            throw e;
        } finally {
            System.out.println("testEx2, finally; return value=" + ret);
            return ret;
        }
    }

    public static void main(String[] args) {
        TestException testException1 = new TestException();
        try {
            testException1.testEx();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

答案是
```
i=2
i=1
testEx2, catch exception
testEx2, finally; return value=false
testEx1, finally; return value=false
testEx, finally; return value=false
```

仔细看上面代码就可以知道异常抛出被testEx2给catch了之后，testEx1等再没有catch到异常，而是直接进入finally。
