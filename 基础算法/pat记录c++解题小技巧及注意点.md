## 小技巧

### 整数前补0
比如23补成0023,输出的时候只需要

``` c++
printf("%04d",&number);
```

### string的使用
在涉及到字符串的时候，不一定要勉强自己使用char数组，也可以include\<string\>来使用。

string overload 許多 operator，包括 +、+=、<、=、[]、<<、>> 等，這些 operator 對字串操作非常方便，因此 assign()、append()、compare() 等函數，除非一些特殊需求，不然一般是用不上。儘量使用 operator，這樣可以讓程序更加易懂。

 - 賦值 = : 將字串指定給另一個字串。
 - **相等比較** == : 比較兩個字串的字元內容是否相同。
 - 串接字串 + : 直接使用 + 運算子來串接字串。
 - 存取字符 []、str.at() : 如字元陣列的操作，at 帶邊界檢查。
 - 字串長度 str.size() : 字串長度。
 - 字串為空 str.empty() : 字串是否為空
 - 字串長度 str.length() : 字串的長度。

string重载了==，不再是比较指针地址了，而是**直接比较string之间的内容**。**赋值也可以直接将c风格字符串(即char数组)直接赋值给string**

用scanf和printf输出时候，只需要调用string的c_str()函数转成c风格字符串即可。

#### 字符串和数字之间的转换

利用sscanf和sprintf函数，这两个都是c语言的函数。sscanf和sprintf两个和scanf和printf不同的是，后者输入源输出初是来自键盘缓冲区，而前者则是自定义源和目的处。

sscanf第一个参数定义从哪里读，sprintf定义第一个参数输出到哪里去。

##### 数字转字符串

``` c++
char str[10];
int a=1234321;
sprintf(str,"%d",a);
```

sprintf显然不能输出东西到int，所以只能反向用sscanf来将字符串转数字。

##### 字符串转数字

``` c++
char str[]="1234321";
int a;
sscanf(str,"%d",&a);
```

##### stringstream类
c++封装的流函数了，有点类似与cin和cout

要include\<sstream\>

``` c++
ostringstream s1;
int i = 22;
s1 << "Hello " << i << endl;
string s2 = s1.str();
cout << s2;

istringstream stream1;
string string1 = "25";
stream1.str(string1);
int i;
stream1 >> i;
cout << i << endl;  // displays 25
```

### c++中的值传递
在java中，由于对象变量是引用，所以容器的操作基本都是引用传递。而c++中，你可以直接操作指针，**容器的添加操作(如vector的push_back)都是直接添加这个对象，而不是传递指针，需要注意**。
