## C++ Union

**“联合”是一种特殊的类**，也是一种构造类型的数据结构。在一个“联合”内可以定义多种不同的数据类型， 一个被说明为该“联合”类型的变量中，允许装入该“联合”所定义的任何一种数据，这些数据共享同一段内存，已达到节省空间的目的。 这是一个非常特殊的地方，也是联合的特征。**另外，同struct一样，联合默认访问权限也是公有的，并且，也具有成员函数**。

union主要是共享内存，分配内存以其 **最大的结构或对象为大小，即sizeof最大的**。在C/C++程序的编写中，当多个基本数据类型或复合数据结构要占用同一片内存时，我们要使用联合体；当多种类型，多个对象，多个事物只取其一时（我们姑且通俗地称其为“n 选1”），我们也可以使用联合体来发挥其长处。


``` c++
union UTest
{
       char   c;
       int    i[ 2 ];
       double var;
};
```

定义以上联合体，sizeof的大小为8.因为int[2]和double的的大小同为8，取最大。

**联合变量可被赋予任一成员值，但每次只能赋一种值， 赋入新值则冲去旧值。**，为Union的一个数据成员赋值会令其他成员变为未定义状态。

另外Union不能被继承，所以不能有虚函数。
