## java变长参数
在 Java 5 中提供了变长参数，允许在调用方法时传入不定长度的参数。变长参数是Java的一个语法糖，本质上还是基于数组的实现。

``` java
void foo(String... args);
void foo(String[] args);

```

变长参数在编译为字节码后，在方法签名中就是以数组形态出现的。这两个方法的签名是一致的，不能作为方法的重载。如果同时出现，是不能编译通过的。可变参数可以兼容数组，反之则不成立。

``` java
public void foo(String...varargs){}

foo("arg1", "arg2", "arg3");

//上述过程和下面的调用是等价的
foo(new String[]{"arg1", "arg2", "arg3"});

```

### 使用规则

#### 优先匹配固定参数
``` java
void foo(String arg1, String arg2) {
    System.out.println("invoke first method!");
}

void foo(String... args) {
    System.out.println("invoke second method!");
}

foo("arg1", "arg2"); //invoke first method!
foo("arg1", "arg2", "arg3"); //invoke second method!

```

#### 如果同时匹配多个可变参数，无法编译通过

``` java
void foo(String arg1, String... args) {
    System.out.println("invoke first method!");
}

void foo(String... args) {
    System.out.println("invoke second method!");
}

foo("arg1");//compile error
```

要注意，**可变参数只能有一个，且必须在参数列表最后**

### 坑

#### 使用 Object… 作为变长参数

``` java
public void foo(Object... args) {
    System.out.println(args.length);
}

foo(new String[]{"arg1", "arg2", "arg3"}); //3
foo(100, new String[]{"arg1", "arg1"}); //2

foo(new Integer[]{1, 2, 3}); //3
foo(100, new Integer[]{1, 2, 3}); //2
foo(1, 2, 3); //3
foo(new int[]{1, 2, 3}); //1

```

int[] 无法转型为 Object[], 因而被当作一个单纯的数组对象 ; Integer[] 可以转型为 Object[], 可以作为一个对象数组。

虽然数组是协变(见泛型笔记有相关概念讲述，如果A是B的子类，那么数组A也是数组B的子类)的，但是int属于基础类型，并不是一个对象，所以int数组只能当做一个数组对象来看待，而不能看作是一个Object数组。
