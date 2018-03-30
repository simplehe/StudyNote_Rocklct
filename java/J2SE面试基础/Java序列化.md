## Java序列化
Java平台允许我们在内存中创建可复用的Java对象，但一般情况下，只有当JVM处于运行时，这些对象才可能存在，即，这些对象的生命周期不会比JVM的生命周期更长。但在现实应用中，就可能要求在JVM停止运行之后能够保存指定的对象，并在将来重新读取被保存的对象。Java对象序列化就能够帮助我们实现该功能。

使用Java对象序列化，在保存对象时，会把其状态保存为一组字节，在未来，再将这些字节组装成对象。必须注意地是，对象序列化保存的是对象的"状态"，即它的成员变量。**由此可知，对象序列化不会关注类中的静态变量**。

### Java序列化的应用场景
 - 当你想把的内存中的对象状态保存到一个文件中或者数据库中时候；
 - 当你想用套接字在网络上传送对象的时候；
 - 当你想通过RMI传输对象的时候；

### Serializable接口
对于任何需要被序列化的对象，都必须要实现接口Serializable,它只是一个标识接口，本身没有任何成员，只是用来标识说明当前的实现类的对象可以被序列化.
如果父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口。
如果**被写对象的类型是String，或数组，或Enum，或Serializable，那么就可以对该对象进行序列化**，否则将抛出NotSerializableException。

### 对象的读写
Java类中对象的序列化工作是**通过ObjectOutputStream和ObjectInputStream来完成的**。
使用readObject()|writeObject()方法对对象进行读写操作；对于基本类型，可以使用readInt()|writeInt(), readDouble()|writeDouble()等类似的接口进行读写。


``` java
public class SerializableTest {

    public static void main(String args[]) throws Exception{

        File file = new File("box.out");

        FileOutputStream fos = new FileOutputStream(file);
        ObjectOutputStream out = new ObjectOutputStream(fos);
        Box oldBox = new Box(10,20);
        out.writeObject(oldBox);
        out.close();

        FileInputStream fis = new FileInputStream(file);
        ObjectInputStream in = new ObjectInputStream(fis);
        Box newBox = (Box)in.readObject();
        in.close();
        System.out.println(newBox.toString());

    }
}

```


### transient关键字
如果某实例变量不能或不应该被序列化，就把它标记为transient的变量，这样序列化程序就会把它跳过。
transient的引用变量会以null返回，基本数据类型会以相应的默认值返回。
（**例如：引用类型没有实现Serializable，或者动态数据只可以在执行时求出而不能或不必存储**）

### 关于serialVersionUID
我们可能经常会看到，在一些类中实现了Serializable接口，同时还定义了一个private final static long型的变量，名字叫serialVersionUID。

```
private final static long serialVersionUID = 123456789L
```

现在设想一个场景，如果将对象序列化后，并没有马上反序列化，如将对象保存到文件中，然后修改类的定义，比如在Student类中加一个字段。
然后再进行反序列化，此时会发生什么样的事情。

有时候，**我们不希望每次修改了类的定义后，以前保存的实例就不能够使用了**，这样会导致每次更新程序会丢失很多数据。
有些时候，修改后的类是完全可以兼容旧的类的。

而程序是通过**比较新旧两个类的serialVersionUID来判断是否是不一样的两个类**。所以我们只需要覆盖serialVersionUID就可以做到兼容了。

覆盖serialVersionUID需要满足得条件：

1.static 静态变量 2.final 常量 3.long 长整型 4.名称为serialVersionUID


``` java
public class Student implements Serializable{
    private final static long serialVersionUID = 1L;
}
```

### 序列化存储规则
Java 序列化机制为了节省磁盘空间，具有特定的存储规则，当写入文件的为同一对象时，并不会再将对象的内容进行存储，而只是再次存储一份引用；
序列化到同一个文件时，如第二次修改了相同对象属性值再次保存时候，虚拟机根据引用关系知道已经有一个相同对象已经写入文件，因此只保存第二次写的引用，所以读取时，都是第一次保存的对象。
