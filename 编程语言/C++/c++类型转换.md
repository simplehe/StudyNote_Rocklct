## c++类型转换
在c++语言中，如果两种类型可以相互转换，那么他们就是关联的。

转换当然存在显示转换和隐式转换，隐式转换的时机是如何呢？

 - 大多数表达式中，比int类型小的整形，会提升为较大的整数(**整形提升**)
 - 条件中，非布尔转换成布尔
 - 初始化过程中，右侧转换成左侧。
 - 函数调用时也有可能发生类型转换

### 显式转换
**C风格的强制类型转换(Type Cast)很简单**，不管什么类型的转换统统是：
TYPE b = (TYPE)a。


然而，**C++风格的类型转换提供了4种类型转换操作符**来应对不同场合的应用。

基本语法都是

``` c++
const-name<type>(expression);
```

const-name是以下四种方式

#### 1.static_cast
类似于C风格的强制转换。无条件转换，静态类型转换。用于:

1. 基类和子类之间转换：其中子类指针转换成父类指针是安全的；但**父类指针转换成子类指针是不安全的**。(基类和子类之间的动态类型转换建议用dynamic_cast)
2. 基本数据类型转换。enum, struct, int, char, float等。**static_cast不能进行无关类型（如非基类和子类）指针之间的转换**。
3. 把空指针转换成目标类型的空指针。
4. 把任何类型的表达式转换成void类型。
5. **static_cast不能去掉类型的const、volitale属性(用const_cast)**。


#### 2.const_cast
去掉类型的const或volatile属性。

主要用来强制性地添加或者去掉一个指针变量或引用变量的底层 const 或底层 volatile。底层其实就是最外层的

``` c++
struct SA {
 int i;
 };
 const SA ra;
 //ra.i = 10; //直接修改const类型，编译错误
 SA &rb = const_cast<SA&>(ra);
 rb.i =10;
```

#### 3.reinterpret_cast
仅仅重新解释类型，但没有进行二进制的转换：

1. 转换的类型**必须是一个指针、引用、算术类型、函数指针或者成员指针**。
2. 在比特位级别上进行转换。它**可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针**（先把一个指针转换成一个整数，在把该整数转换成原类型的指针，还可以得到原先的指针值）。但不能将非32bit的实例转成指针。
3. 最普通的用途就是**在函数指针类型之间进行转换**。
4. 很难保证移植性。

``` c++
 int doSomething(){return0;};
 typedef void(*FuncPtr)(); //FuncPtr is 一个指向函数的指针，该函数没有参数，返回值类型为 void
 FuncPtr funcPtrArray[10]; //10个FuncPtrs指针的数组 让我们假设你希望（因为某些莫名其妙的原因）把一个指向下面函数的指针存入funcPtrArray数组：

 funcPtrArray[0] =&doSomething;// 编译错误！类型不匹配，reinterpret_cast可以让编译器以你的方法去看待它们：funcPtrArray
 funcPtrArray[0] = reinterpret_cast<FuncPtr>(&doSomething); //不同函数指针类型之间进行转换
```

#### 4.dynamic_cast
有条件转换，动态类型转换，运行时类型安全检查(转换失败返回NULL)：
1. 安全的基类和子类之间转换。
2. 必须要有虚函数。
3. 相同基类不同子类之间的交叉转换。但结果是NULL。

``` c++
1 class BaseClass {
2 public:
3 int m_iNum;
4 virtualvoid foo(){}; //基类必须有虚函数。保持多台特性才能使用dynamic_cast
5 };
6
7 class DerivedClass: public BaseClass {
8 public:
9 char*m_szName[100];
10 void bar(){};
11 };
12
13 BaseClass* pb =new DerivedClass();
14 DerivedClass *pd1 = static_cast<DerivedClass *>(pb); //子类->父类，静态类型转换，正确但不推荐
15 DerivedClass *pd2 = dynamic_cast<DerivedClass *>(pb); //子类->父类，动态类型转换，正确
16
17 BaseClass* pb2 =new BaseClass();
18 DerivedClass *pd21 = static_cast<DerivedClass *>(pb2); //父类->子类，静态类型转换，危险！访问子类m_szName成员越界
19 DerivedClass *pd22 = dynamic_cast<DerivedClass *>(pb2); //父类->子类，动态类型转换，安全的。结果是NULL
```
