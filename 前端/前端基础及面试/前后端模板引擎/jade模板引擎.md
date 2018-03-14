## jade模板引擎

<a>http://naltatis.github.io/jade-syntax-docs/#basics</a>

 jade是使用JavaScript实现，可供nodejs使用的高性能模板引擎（性能高不高，有些争议。姑且称之为高性能）

简而言之就是讲jade模板编译成html

从实际意义上来说，Jade是为了提高前端开发人员的效率而产生的。Jade中，空格、换行、缩进都是有意义的，由这些决定了标签和内容的嵌套关系。

### 用法
标签就是一个简单单词

#### 标签处理
html

会被转换成

```
<html></html>
```

标签可以添加id，比如

div#container

对标签添加class也同理，div.class1

#### 内容
标签里的内容，以一个空格跟在标签名后，比如

p wahoo!

这会被渲染成
```
<p>wahoo!</p>
```

大段数据的时候

```
p
  | foo bar baz
  | rawr rawr
  | super cool
  | go jade go
```

渲染成
 
```
<p>foo bar baz rawr.....</p>
```

这里还有一种选择，可以使用 . 来开始一段文本块
```
p.
  foo asdf
  asdf
   asdfasdfaf
   asdf
  asd.
```

会被渲染为
```
<p>foo asdf
asdf
  asdfasdfaf
  asdf
asd
.
</p>
```

只包含文本的标签，比如 \<script\>,\<style\>, 和 \<textarea\> 不需要前缀 | 字符



####　内容渲染
内容用#{}来替换，比如

```
#user #{name} &lt;#{email}&gt;
```

会被渲染为

```
<div id="user">tj &lt;tj@vision-media.ca&gt;</div>
```


### 变量
模板引擎真正的强大的地方是可以实现函数式开发。


```
- var cs = 'UTF-8'
meta(charset='#{cs}')

//编译出来的结果
<meta charset="UTF-8">
```
前面加横杠符号，可以声明变量。


\#{}和!{}都可以作为变量替换的标志，然而#{}的话会进行html转义,!{}则不会。

除了用#{}和!{}外，也可以在标签后面紧接=等号（不转义用!=）来输出变量。例如：

### 语句
jade支持js的一些语句，比如if-else,unless等

```
- var author = 'Jack';
if author
  p 作者：#{author}
else
  p 无作者

//编译出来的结果
<p>作者：Jack</p>
```

for语句可以的，不过for的前面要加横杠

### minxin
类似于函数一样的东西，可以重用代码。用mixin来封装代码也比较简单。

```
mixin sayHi
  p Hi
+sayHi

//编译出来的结果
<p>Hi</p>
```

调用的时候要在mixin前面用+号

