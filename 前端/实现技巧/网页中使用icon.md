## 网页中插入icon小图标
有好几个方法使用icon小图标
比如

1. CSS Sprite
2. font+HTML
3. font+CSS

### 
CSS Sprite技术见另一篇笔记

### font+HTML

***使用字体用HTML代码以文本的形式直接在网页中画icon小图标。***

#### 好处
使用图标字体可大大减少图标维护工作量。 

1. 灵活性：轻松地改变图标的颜色或其他CSS效果。 
2. 可扩展性：改变图标的大小，就像改变字体大小一样容易。 
3. 矢量性：图标是矢量的，与像素无关。缩放图标不会影响清晰度。 
4. 兼容性：字体图标支持所有现代浏览器（包括糟糕的IE6）。 
5. 本地使用：通过添加定制字体到你的本地系统，你可以在各种不同的设计和编辑应用程序中使用它们。


#### font-face模块
首先必须介绍这个font-face模块，这是css3中的一个模块，主要是用于嵌入自定义字体，这样的自定义字体可以用于各种浏览器之中，甚至ie浏览器都支持这样的自定义字体。其中一些字体定义格式，还有压缩算法，使得传输效率非常高。

使用方法

``` css
@font-face{
font-face:"family-name";  /*自定义字体名称*/
src:url("../font/XXX.eot");  /*解决IE9兼容模式下的兼容性问题*/
src:url("../font/XXX.eot?") format("embedded-opentype"),  /*在eot加?或者?iefix，以解决字体文件在IE下加载失败的问题*/
    url("../font/XXX.woff") format("woff"),
    url("../font/XXX.ttf") format("trueytype"),
    url("../font/XXX.syg") format("svg");
font-weight:normal;  /*定义字体粗细*/
font-style:normal;  /*定义字体样式*/
｝
```

这里定义font face导入了本地下载的字体文件，做一些基本的定义，定义这个字体文件的名字。定义好之后，就可以在css中使用。

比如

``` css
.icon｛              /*按照一般的字体来写样式即可*/
font-family:"family-icon";
font-style:normal;
font-weight:normal;
font-size:20px;
-webkit-font-smoothing:antialiased;  /*用于webkit引擎中的字体抗锯齿属性*/
-moz-osx-font-smoothing:graycale  /*用于Mac OS系统和火狐浏览器中的字体抗锯齿属性*/
｝
.example{
color:#fff;  /*可轻松改变图标颜色*/
font-size:24px;  /*改变图标的大小*/
}

/*
#iefix有何作用？
IE9 之前的版本没有按照标准解析字体声明，当 src 属性包含多个 url 时，它无法正确的解析而返回 404 错误，而其他浏览器会自动采用自己适用的 url。因此把仅 IE9 之前支持的 EOT 格式放在第一位，然后在 url 后加上 ?，这样 IE9 之前的版本会把问号之后的内容当作 url 的参数。至于 #iefix 的作用，一是起到了注释的作用，二是可以将 url 参数变为锚点，减少发送给服务器的字符。
为何有两个src？
绝大多数情况下，第一个 src 是可以去掉的，除非需要支持 IE9 下的兼容模式。在 IE9 中可以使用 IE7 和 IE8 的模式渲染页面，微软修改了在兼容模式下的 CSS 解析器，导致使用 ? 的方案失效。由于 CSS 解释器是从下往上解析的，所以在上面添加一个不带问号的 src 属性便可以解决此问题。
*/
```



然后在html中调用对应的代码输出。

``` html
<!-- 将icon图标输出时，需要在icon的16进制编码前加上&#x -->
<i class="icon example">&#xf048;</i>
```

#### 几种字体文件格式
- EOT(Embedded OpenType Fonts)：微软开发的用于嵌入网页中的字体，**IE专用字体**。
- WOFF(The Web Open Font Format)：Web字体中最佳格式，是一个开放的TrueType/OpenType的压缩版本，2009年被开发，如今被W3C组织推荐标准。WOFF字体通常比其它字体加载的要快些，因为使用了OpenType (OTF)和TrueType (TTF)字体里的存储结构和压缩算法。这种字体格式还可以加入元信息和授权信息。这种字体格式有君临天下的趋势，因为所有的现代浏览器都开始支持这种字体格式。
- TTF(TrueType Fonts)：1980s，由Microsoft和Apple联合开发的一套字体标准，是Mac OS和WIN操作系统中最常见的一种字体。
- SVG(SVG Fonts)：用于SVG字体渲染的一种格式，是由W3C制定的开放标准的图形格式。

#### 一些icon字体图标网站
- http://fontawesome.io/
- https://icomoon.io/
- http://www.bootcss.com/p/font-awesome/
- http://unicode-table.com/cn/
- https://design.google.com/icons/
- 如何自制字体图标：http://flowerboys.cn/font/article/fontsarticle/how-to-turn-your-icons-into-a-web-font.html


### font+CSS
除了导入字体之后，还可以导入字体对应的css文件。最后直接调用样式就可以了。

**主要使用:before伪元素、after伪元素、content属性**

``` css
/*@font-face属性、.icon样式与上面方法无异*/
.example:before{
content:"\f048"  /*需要在16进制编码前加上\进行转译*/
}
```

``` html
<i class="icon example"></i>
```