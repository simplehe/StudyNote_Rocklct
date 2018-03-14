## meta标签
<meta> 元素可提供有关页面的元信息（meta-information），比如针对搜索引擎和更新频度的描述和关键词。

<meta> 标签位于文档的头部，不包含任何内容。<meta> 标签的属性定义了与文档相关联的名称/值对。

元数据总是以名称/值的形式被成对传递的。

meta标签的组成：meta标签共有两个属性，它们分别是http-equiv属性和name属性，不同的属性又有不同的参数值，这些不同的参数值就实现了不同的网页功能。

### name属性
name属性主要用于描述网页，与之对应的属性值为content，content中的内容主要是便于搜索引擎机器人**查找信息**和**分类信息**用的。

语法格式为
``` html
<meta name="参数"content="具体的参数值">。 
```

比如
``` html
<meta name="keywords"content="meta总结,html meta,meta属性,meta跳转"> 
```

其他还有诸如name=robots，用来告诉搜索机器人哪些页面需要索引，哪些页面不需要索引。属性值有all,none,index,noindex,follow,nofollow。具体看文档


### http-equiv属性
http-equiv顾名思义，相当于http的文件头作用，它可以向浏览器传回一些有用的信息，以帮助正确和精确地显示网页内容，与之对应的属性值为content，content中的内容其实就是各个参数的变量值。

**一般来说，在网页中指定的模式，高于在服务器中用HTTP header所指定的模式**  

所以 **meta tag > http header**

``` html
http-equiv顾名思义，相当于http的文件头作用，它可以向浏览器传回一些有用的信息，以帮助正确和精确地显示网页内容，与之对应的属性值为content，content中的内容其实就是各个参数的变量值。
```

经典的一些参数

比如expires，设定到期时间。

Pragma，content有cache和nocache，可以禁止浏览器从本地计算机的缓存页面中访问页面内容。

content-Type，设定页面所用的字符集。

**注意html引入了一个新的书写方式**

如charset
``` html
<head>
<meta charset="UTF-8">
</head>
```

省去了形式
``` html
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
```

可以有效的减少代码量

#### X-UA-Compatible
``` html
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

X-UA-Compatible这个属性，规定页面应该用什么样的模式来渲染。

首先要知道这个属性值对于IE8以下的浏览器都是不识别的，下面来看一些例子

``` html
<meta http-equiv="X-UA-Compatible" content="IE=7">  
#以上代码告诉IE浏览器，无论是否用DTD声明文档标准，IE8/9都会以IE7引擎来渲染页面。  
<meta http-equiv="X-UA-Compatible" content="IE=8">  
#以上代码告诉IE浏览器，IE8/9都会以IE8引擎来渲染页面。  
<meta http-equiv="X-UA-Compatible" content="IE=edge">  
#以上代码告诉IE浏览器，IE8/9及以后的版本都会以最高版本IE来渲染页面。  
<meta http-equiv="X-UA-Compatible" content="IE=7,IE=9">  
<meta http-equiv="X-UA-Compatible" content="IE=7,9">  
<meta http-equiv="X-UA-Compatible" content="IE=Edge,chrome=1">
#以上代码IE=edge告诉IE使用最新的引擎渲染网页，chrome=1则可以激活Chrome Frame.
```

注意，设置chrome=1，则是激活谷歌渲染的意思，比如有的浏览器用的ie内核，确也有装谷歌渲染的插件，那么这样设置，就是让你用谷歌内核渲染的意思。