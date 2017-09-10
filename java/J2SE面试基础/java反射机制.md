## java反射机制
要让Java程序能够运行，就得让Java类被Java虚拟机加载。Java类如果不被Java虚拟机加载就不能正常运行。正常情况下，我们运行的所有的程序在编译期时候就已经把那个类被加载了。

Java的反射机制是在编译时并不确定是哪个类被加载了，而是在程序运行的时候才加载、探知、自审。使用的是在编译期并不知道的类。这样的编译特点就是java反射。

### Class对象的使用和反射
JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的**所有属性和方法**；对于任意一个对象，都能够调用它的任意一个方法和属性；**这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制**。

当java类编译成class文件后并不会立即被加载到内存，而是在它的的静态成员第一次被访问时才被加载(这么看来，java类的默认构造方法也是静态的！)

调用反射api的例子
``` java
Class c = Cat.class;
        Field[] fields = c.getDeclaredFields();
        for (Field field : fields){
            System.out.println(field);
        }
```

### setAccessible 调用私有域和方法
**Method、Field和Constructor类共同继承了AccessibleObject类**，该基类有两个setAccessible方法能在**运行时压制Java语言访问控制检查(Java language access control checks)，从而能任意调用被私有化保护的方法、域和构造方法**

java代码中，常常将一个类的成员变量置为private

在类的外面获取此类的私有成员变量的value时，要注意

假定有如下代码

``` java
public class Main {

    public static void main(String[] args) throws Exception {
        Class clazz = Class.forName("com.test.accessible.AccessibleTest");
        AccessibleTest at = new AccessibleTest();
        at.setId(1);
        for (Field f : clazz.getDeclaredFields()) {
            f.setAccessible(true);//AccessibleTest类中的成员变量为private,故必须进行此操作
            System.out.println(f.get(at));//获取当前对象中当前Field的value
        }

    }

}
```

我们假定定义了一个AccessibleTest类，他里面的有个private int id变量。上述代码首先setid为1了，然后这里要注意，for循环可以获得AccessibleTest类的各个域，也就是实例变量。但是id是private的，理应不可以由外面访问，所以这里才有了setAccessible为true。压制了语言访问控制检查，从而可以获得这个private filed。

如果没有在获取Field之前调用setAccessible(true)方法，异常：

```
java.lang.IllegalAccessException:
Class com.test.accessible.Main
can not access
a member of class com.test.accessible.AccessibleTest
with modifiers "private"
```
