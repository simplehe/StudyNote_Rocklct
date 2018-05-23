## c++类
这里记录一些c++类的知识点。

### class和struct
在c++中，struct定义出来的也是类，它们唯一的区别就是默认的访问权限不太一样。

如果我们使用struct关键字，则定义在第一个访问说明符之前的成员(默认权限)是public。

如果使用class关键字，默认成员则是private。

### 构造函数

#### 默认构造函数=default
我们希望有多个构造函数，但是也需要一个不接受任何实参的默认构造函数。

``` c++
struct a{
  a()=default;
};
```

如果=default定义在类内部，则默认是内联的。

如果不在类内定义任何构造函数，编译器会自动生成一个**合成的默认构造函数**

#### 构造函数初始化列表

``` c++

class a{

  a(int dd,int ii):d(dd),i(dd*ii){}
  a():d(0),i(0){}
  double d;
  int i;
}

```

冒号与花括号之间定义了初始化值列表，用来给成员赋初值。

如果成员对象是const，那必须用初始化列表初始化，否则会报错。

``` c++
class ConstRef{
public:
  ConstRef::ConstRef(int ii):ci(ii) {}
private:
  const int ci;
}

```

此外，**初始化的顺序和初始化列表的定义顺序无关，但是跟成员变量的定义顺序有关**。一般来说顺序是没什么关系，但一旦涉及一个变量初始化要用另外一个变量来初始化，顺序就要注意了。

``` c++
class X{
  int i;
  int j;
public:
  X(int val):j(val),i(j){}//wrong！i init first
};
```

**上面例子会报错**，需要注意，虽然看上去j比i先初始化，事实正好相反。


#### 拷贝构造函数
如果一个构造函数的第一个参数是自身类型的引用，且任何额外参数都有默认值，则此构造函数是**拷贝构造函数**。

``` c++
class Foo{
public:
  Foo(); //默认构造函数
  Foo(const Foo&);// 拷贝构造函数
}
```

如果我们不定义，编译器也会生成**合成拷贝构造函数**，它会将(非static)成员逐个拷贝到正在创建的对象中，成员对象之间的拷贝也会递归使用拷贝构造函数去赋值。

拷贝构造函数会在以下时候调用

 - 定义变量初始化时(比如 `string s="sss";`)
 - 对象的实参形参传递
 - 花括号初始化数组或者聚合类

#### 拷贝赋值
在给对象赋值的时候，会调用拷贝赋值运算符，如果没有定义，编译器会像创建拷贝构造函数一样生成一个合成的。


我们可以重载=运算符来改变赋值运算

``` c++
class Foo {
  public:
  Foo& operator=(const Foo&);
}
```

### 移动构造函数
和拷贝构造函数区别就是不用拷贝实参，而是直接移动，性能上会有提升。格式是class(const class \&\&) noexcept; class\&\&是右值引用，可见左右值笔记。

移动实现的是对象值**真实的转移**（源对象到目的对象）：源对象将丢失其内容，其内容将被目的对象占有。移动操作的发生的时候，是当移动值的对象是**右值**的时候。

``` c
MyClass (MyClass&&);             // move-constructor
MyClass& operator= (MyClass&&);  // move-assignment
```



### 析构函数
与构造函数执行相反的操作，用以释放对象资源。

``` c++
class Foo{
public:
  ~Foo();
}
```

**与构造函数不同，析构函数是先执行函数体，然后再按照初始化的顺序逆序把成员变量释放。**

析构函数的调用时机

 - 变量离开作用域
 - 对象销毁
 - 容器被销毁，元素分别析构
 - 指针被delete，指向对象也被销毁
 - 临时对象表达式结束时

如果不定义析构函数，编译器也会合成一个默认的，合成析构函数。

### 成员
c++在调用函数的时候**引入了this**这个额外参数

一个对象的this指针并不是对象本身的一部分，不会影响sizeof(对象)的结果。this作用域是在类内部，当在类的非静态成员函数中访问类的非静态成员的时候，编译器会**自动将对象本身的地址作为一个隐含参数传递给函数**。也就是说，即使你没有写上this指针，编译器在编译的时候也是加上this的，它作为非静态成员函数的隐含形参，对各成员的访问均通过this进行。

比如我们调用

``` c++
A aa;
aa.a()
```

则编译器相当于执行了

``` c++
A::a(&aa);
```

相当于自动传入一个实例的指针。

#### const成员函数
定义成员函数的时候，可以在函数后加const关键字。

``` c++
std::string isbn() const { return this->book;}
```

这里加const关键字的目的，就是改变函数中this指针的类型。

我们知道默认情况下，this指针都是指向实例，而且从上面笔记也可以知道this指针隐式遵循初始化原则。假如我们有一个类A，那么A的成员函数中，**this指针的默认类型为 A* const**，根据const笔记，我们知道这是一个常量指针，也就是说指针本身是不可变的，但是我们可以改变指针指向的内容，也就是说我们可以写set成员函数来给成员对象赋值。

如果我们给函数后面加一个const，那就是改变了这个this指针的属性，使其变成const A\* const，是一个双重const指针，既不能改变自身也不能改变其指向的属性。用const在最后修饰的函数，也叫 **常量成员函数**。

常量成员函数不能改变调用它的对象的内容，但是可以读取。


C++允许成员方法基于基本的const类型进行重载

``` c++
class A{
  void a() {} //fun1
  void a() const {} //fun2
}

A a1;
const A a2;
a1.a(); //fun1
a2.a(); // fun2
```

由以上代码可知，虽然调用a方法的参数和类型都相同，只是在尾部添加了const，但c++允许通过这个方法重载。所以当const变量a2调用方法时，调用的是第二个基于const的成员方法。

#### 类内成员函数默认内联
定义在类内的成员函数是默认inline的，如果在类外部定义成员函数，最好显示用inline定义。


#### 友元函数
**类可以允许其他类或者函数访问他的非共有成员**，方法是令其他类或者函数成为它的友元friend。


``` c++
class Student{
public:
    Student(char *name, int age, float score);
public:
    friend void show(Student *pstu);  //将show()声明为友元函数
private:
    char *m_name;
    int m_age;
    float m_score;
};

//非成员函数
void show(Student *pstu){
    cout<<pstu->m_name<<"的年龄是 "<<pstu->m_age<<"，成绩是 "<<pstu->m_score<<endl;
}

int main(){
    Student stu("小明", 15, 90.6);
    show(&stu);  //调用友元函数
    Student *pstu = new Student("李磊", 16, 80.5);
    show(pstu);  //调用友元函数
    return 0;
}
```



#### 类型成员

看以下代码

``` c++
class Screen{
  public:
    typedef std::string::size_type pos;
  private:
    pos cursor = 0;

}
```

public处定义了一个类型成员，封装细节。
