## c++动态加载
在静态库与动态库的笔记当中，我们知道了两种的区别。现在两讲讲在操作系统中，我们如何具体地动态加载库文件。

### Linux动态加载

虽然Linux下还 **没有** 直接的机制可以在运行时加载C++类，但是有一个直接在运行时加载C库的机制：dl函数dlopen，dlsym，dlerror和dlclose。这组函数提供了访问动态链接器ld的方法。

``` c++
void *dlopen(const char * filename, int flag);

void *dlsym(void *handle, char*symbol);

const char *dlerror();

int dlclose(void *handle);
```

如上，这些函数都在dlfcn头文件中。

 **dlopen** 函数通过文件名filename打开so文件，文件中的符号通过 **dlsym** 函数读取。参数flag可以取下面的值：`RTLD_LAZY`和`RTLD_NOW`。如果flag设置为RTLD_LAZY，那么dlopen会不解析任何符号就返回。如果flag设置为RTLD_NOW，那么dlopen会尝试解析文件中所有未定义的符号。如果出现解析错误，函数调用失败，返回NULL。

 也就是说dlsym获取执行了dlopen函数的对象文件中的 **函数地址**。

**dlerror** 解析失败的原因。dlsym函数用于读取库中函数（或其他符号）的指针。handle是指向被引用项的指针，symbol是被引用项在实际保存文件中的字符串名字。

#### Example

``` c++
#include <dlfcn.h>

typedef long (*PFN_TEST)(const char* szName, int nAge);
PFN_TEST g_Test = NULL;

void* handle = dlopen("/path/to/so", RTLD_LAZY);
 if(!handle)
 {        
         printf("ERROR, Message(%s).\n", dlerror());
         return -1;
 }

 g_Test = (PFN_TEST)dlsym(handle, "Test");
 char* szError = dlerror();
 if(szError != NULL)
 {
     printf("ERROR, Message(%s).\n", szError);
     dlclose(handle);
     return -1;
 }
 if(g_Test != NULL)
 {
     g_Test ("wjshan", 0808);
 }
 dlclose(handle);
 g_Test = NULL;
 return 0;
```

如上代码，先引入头文件，然后定义一个函数指针类型PFN_TEST,定义一个实例g_Test.

用dlopen打开动态库so文件，指针不为空的情况下用dlsym去找Test函数(symbol).如果这个函数指针不为空，那么久执行。最后用dlclose关闭文件。
