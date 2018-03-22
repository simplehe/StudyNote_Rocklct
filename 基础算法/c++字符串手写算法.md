## 面试题之strcpy/strlen/strcat/strcmp的实现

### 字符串拷贝strcpy

函数strcpy的原型是
`char* strcpy(char* des , const char* src)`，des 和 src 所指内存区域不可以重叠且 des 必须有足够的空间来容纳 src 的字符串。

``` c++
#include <assert.h>
#include <stdio.h>

char* strcpy(char* des, const char* src)
{
	assert((des!=NULL) && (src!=NULL));
	char *address = des;
	while((*des++ = *src++) != '\0')
		;
	return address;
}
```

要知道 strcpy 会拷贝’\0’，还有要注意：

 - 源指针所指的字符串内容是不能修改的，因此应该声明为 const 类型。

 - 要判断源指针和目的指针为空的情况，思维要严谨，这里使用assert（见文末）。

 - 要用一个临时变量保存目的串的首地址，最后返回这个首地址。

 - 函数返回 char* 的目的是为了支持链式表达式，即strcpy可以作为其他函数的实参。


### 字符串长度strlen


``` c++
#include <assert.h>
#include <stdio.h>

int strlen(const char* str)
{
	assert(str != NULL);
	int len = 0;
	while((*str++) != '\0')
		++len;
	return len;
}
```


### 字符串连接strcat

``` c++
#include <assert.h>
#include <stdio.h>

char* strcat(char* des, const char* src)   // const表明为输入参数
{
	assert((des!=NULL) && (src!=NULL));
	char* address = des;
	while(*des != '\0')  // 移动到字符串末尾
		++des;
	while(*des++ = *src++)
		;
	return address;
}
```


### 字符串比较strcmp

函数strcmp的原型是`int strcmp(const char *s1,const char *s2)`

``` c++
#include <assert.h>
#include <stdio.h>

int strcmp(const char *s1,const char *s2)
{
	assert((s1!=NULL) && (s2!=NULL));
    while(*s1 == *s2)
    {
        if(*s1 == '\0')
            return 0;

        ++s1;
        ++s2;
    }
    return *s1 - *s2;
}
```
