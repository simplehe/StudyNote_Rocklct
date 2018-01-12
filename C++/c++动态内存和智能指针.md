## C++动态内存和智能指针
动态内存通过new/delete来完成，正如c++内存管理笔记里所说，它会自动调用对象的构造函数，然后返回一个指针，最后delete的时候调用析构函数。

动态内存容易出现问题，确保在正确时间释放内存及其困难。有时候我们忘记释放内存(new以后不显式delete对象，则只有指针被回收)，则会产生内存泄漏。或者提前就释放了它，会产生非法引用内存的指针。

新的标准库提供了**两种智能指针smart pointer类来管理动态对象**，智能指针会负责自动释放所指向的对象
 - shared_ptr 允许多个指针指向同一个对象
 - unique_ptr 独占所指向的对象
 - weak_ptr则是一种弱引用，指向shared_ptr所管理的对象

三种类型都定义在memory头文件当中，还有一种auto_ptr已经被弃用。

### shared_ptr
智能指针也是模板，定义的时候要提供类型信息。

shared_ptr的目标非常简单：多个指针可以同时指向一个对象，当**最后一个shared_ptr离开作用域时**，内存才会自动释放(自动调用析构函数)。

#### make_shared函数
头文件memory中定义的一个标准库函数，是**最安全的分配和使用动态内存的方式**

``` c++
shared_ptr<int> p = make_shared<int>(42);
```

上面的代码创建了一个shared_ptr,指向一块内存，该内存包含一个整数，以及引用计数1.如果通过p再创建一个shared_ptr,引用计数就会变成2. 该计数被称为强引用(strong reference)，除此之外，shared_ptr还有另外一种引用计数叫做弱引用(weak reference)

通过调用use_count()可以得到引用计数， 据此你能找到shared_ptr的数量。当debug的时候，可以通过观察shared_ptr中strong_ref的值得到引用计数。

此外，shared_ptr默认调用delete释放关联的资源。如果用户采用一个不一样的析构策略时，他可以自由指定构造这个shared_ptr的策略。

``` c++
void main( )
{
 shared_ptr<Test> sptr1( new Test[5],
        [ ](Test* p) { delete[ ] p; } );
}
```

在此场景下，shared_ptr指向一组对象，但是当离开作用域时，默认的析构函数调用delete释放资源。实际上，我们应该调用delete[]来销毁这个数组。

### unique_ptr
unique_ptr也是对auto_ptr的替换。unique_ptr遵循着独占语义。在任何时间点，资源只能**唯一地被一个unique_ptr占有**。当unique_ptr离开作用域，所包含的资源被释放。如果资源被其它资源重写了，之前拥有的资源将被释放。所以它保证了他所关联的资源总是能被释放。

unique_ptr提供了创建数组对象的特殊方法，当指针离开作用域时，调用delete[]代替delete。当创建unique_ptr时，这一组对象被视作模板参数的部分。这样，程序员就不需要再提供一个指定的析构方法，如下：

``` c++
unique_ptr<int[ ]> uptr( new int[5] );
```

### Weak_Ptr
weak_ptr 拥有共享语义（sharing semantics）和不包含语义（not owning semantics）。这意味着，weak_ptr可以共享shared_ptr持有的资源。所以可以从一个包含资源的shared_ptr创建weak_ptr。

``` c++
auto p = make_shared<int>(42);
weak_ptr<int> wp(p);
```

弱共享不会改变对象的引用计数，所以只要强引用全部释放了，弱引用也不能避免对象被回收。

weak_ptr不支持普通指针包含的\*，->操作。它**并不包含资源所以也不允许程序员操作资源**。既然如此，我们如何使用weak_ptr呢？

答案是从weak_ptr中创建shared_ptr然后再使用它。

``` c++
if(shared_ptr<int> np = wp.lock()){
  ......
}
```

由于我们使用弱引用的时候，对象不一定还存在，所以我们必须先调用lock，检查对象是否还存在，如果还存在，就返回一个shared_ptr强引用，这就保证了有至少一个强引用存在，使用的时候对象不会被回收。
