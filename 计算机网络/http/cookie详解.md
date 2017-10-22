## cookie详解
简单地说，cookie 就是浏览器储存在用户电脑上的一小段**文本文件**。cookie 是纯文本格式，不包含任何可执行的代码。一个 Web 页面或服务器告知浏览器按照一定规范来储存这些信息，并在随后的请求中将这些信息发送至服务器，Web 服务器就可以使用这些信息来识别不同的用户。大多数需要登录的网站在用户验证成功之后都会设置一个 cookie，只要这个 cookie 存在并可以，用户就可以自由浏览这个网站的任意页面。再次说明，cookie 只包含数据，就其本身而言并不有害。

Web 服务器通过发送一个称为 Set-Cookie 的 HTTP 消息头来创建一个 cookie，Set-Cookie 消息头是一个字符串，其格式如下（中括号中的部分是可选的）：

```
Set-Cookie: value[; expires=date][; domain=domain][; path=path][; secure]
```

更详细些的设置可以看http协议详解笔记。

当存在一个 cookie，并允许设置可选项，该 cookie 的值会在随后的**每次请求中被发送至服务器**，cookie 的值被存储在名为 **Cookie 的 HTTP 消息头中**，并且**只包含**

了 cookie 的值，忽略全部设置选项。例如：

```
Cookie: value
```

### 基本工作原理
首先必须明确一点，存储cookie是浏览器提供的功能。cookie 其实是存储在浏览器中的**纯文本**，浏览器的安装目录下会专门有一个 **cookie 文件夹来存放各个域下设置的cookie**。

当网页要发http请求时，浏览器会先检查是否有相应的cookie，有则自动添加在request header中的cookie字段中。这些是浏览器自动帮我们做的，而且每一次http请求浏览器都会自动帮我们做。这个特点很重要，因为这关系到“什么样的数据适合存储在cookie中”。

 localStorage 出现之前，cookie被滥用当做了存储工具。什么数据都放在cookie中，即使这些数据只在页面中使用而不需要随请求传送到服务端。当然cookie标准还是做了一些限制的：**每个域名下的cookie 的大小最大为4KB**，**每个域名下的cookie数量最多为20个**（但很多浏览器厂商在具体实现时支持大于20个）。


### Cookie设置时的domain选项
domain，指定了 cookie 将要被发送至哪个或哪些域中。默认情况下，domain 会被设置为创建该 cookie 的页面所在的域名，所以当给**相同域名发送请求时**该 cookie 会被发送至服务器。

像 Yahoo! 这种大型网站，都会有许多 name.yahoo.com 形式的站点（例如：my.yahoo.com, finance.yahoo.com 等等）。将一个 cookie 的 domain 选项设置为 yahoo.com，就可以将该 cookie 的值发送至**所有这些站点**。浏览器会把 domain 的值与请求的域名做一个**尾部比较**（即从字符串的尾部开始比较），并将匹配的 cookie 发送至服务器。

zwlj：因此想要所有二级或者三级子域名都能访问一个cookie，那就必须set-cookie的时候设置顶级域名。

domain 选项的值必须是发送 Set-Cookie 消息头的主机名的一部分，例如我不能在 google.com 上设置一个 cookie，因为这会产生安全问题。**不合法的 domain 选择将直接被忽略**。

zwlj:也就是说，b网站给我们设置的cookie，浏览器不会允许你发送给a。

### 第一方与第三方cookie
Cookie通常可以分为两类，第一方Cookie和第三方Cookie。都是网站在客户端上存放的一小块数据。他们都由某个域存放，只能被这个域访问。他们的区别其实并不是技术 上的区别，而是使用方式上的区别。

第一方cookie就是a网站域名下设置，且只能用于a网站的cookie，而第三方就是**b域名下设置，但是却可以被a域名读取的cookie**。

#### 第三方cookie的应用
我们用第三方cookie实现许多事情。比如说单点登录，在A网站进行了登录，之后在B网站就自动登陆了。这是由于B网站用跨域手段发请求去请求询问了用户在A网站的cookie。

同时，第三方cookie也**大量应用于广告当中**。比如我访问了某一个广告网站，这个广告就直接在我们浏览器设置了cookie，而且这个cookie还是给别的网站设置的，比如set domain为baidu.com。这样我们在访问百度的时候，就会看到广告经常跟着我们，这就是因为百度收取了推广费，读取了这些第三方cookie。

因此实现读取第三方cookie，有主动和被动两种形式。一种是**我用跨域手段，去向第三方服务器请求cookie**，**一种是，我直接设置cookie给第三方网站用(浏览器要接受)**。


### cookie跨域的具体实现

#### 跨域手段实现获取第三方cookie
跟踪第三方cookie，意思就是用**跨域获取数据的手段去获取别的网站的cookie**。

这种方法技巧就很多了，比如JSONP，在相关笔记里我们已经了解其中原理。

比如说单点登陆的问题，我们在a网站上登陆了，在b网站上也会自动登陆。

原因是，我们在a登陆以后，a给我们设置了cookie。然后我们登陆b，b这时就会利用各种手段获取a的cookie，比如用JSONP。b网站的script标签里链接一个js文件或者说链接一个网址，指向a。

我们知道script标签不受同源策略影响，这时候浏览器就会根据这个b网站上的script标签的指示去a询问cookie。这里注意这个询问的过程，浏览器是会带上a的cookie的。

算成http请求的流程就是：b的response根据script访问a-->访问a的"获取cookie服务"返回cookies字符串(访问a的域名时自然带上了a的cookie,从而a知道你是谁)-->返回了json字符串(cookie)，调用回调方法，重新设置cookie(给b).

当然我们还可以用iframe的方法设置cookie，a网站中嵌入b网站去获取cookie，获取来之后由于ab同处于一个框架之中，所以可以用window.name或者sendMessage方法共享。


#### 利用P3P突破跨域限制
当然，我们也有需求是在访问B网站的时候直接给A设置cookie。这非常直接，访问B的时候直接返回一个set-cookie:domain=A,强硬的给A设置cookie。大部分浏览器都允许这么做，但是IE严格规定不行，一定要设置P3P头。

##### P3P
这里引出了P3P这个概念

**P3P 全称 Platform for Privacy Preferences，隐私设定平台规范**。这个规范极其复杂，若要讲清楚，天都黑了一半。**简言之，就是网站向浏览器声明自己的隐私政策**，比如网站是否搜集访问者的个人信息，设置 cookie 的用途等等。浏览器会**依据设置**，决定在第三方请求的条件下是否**接受网站的 cookie**。


zwlj：**p3p解决的问题是，允许浏览器让b网站设置a网站的cookie，比如我b网站返回了一个http报文，里面有个Set-Cookie里面竟然set了domain为a网站的cookie。严格的IE浏览器不允许你这么做，所以要加p3p头。加了p3p头以后，浏览器才决定是否要接受你这个cookie**


所以我们需要做的就是在服务端里，给网页设置一个p3p header

比如jsp代码

``` java
xmlhttp.setRequestHeader(
"P3P",'CP="CURa ADMa DEVa PSAo PSDo OUR BUS UNI PUR INT DEM STA PRE COM NAV OTC NOI DSP COR"');
```

这些 IDC DSP 什么的是啥意思啊？

这些标签就是 P3P 所规定的了，例如 NOI 表示不搜集可识别用户的资料，ADM 表示信息搜集会用于网站管理……

中文清单:

<a>https://renjin.blogspot.hk/2008/02/p3p.html</a>

除了传送 P3P http header，还可以通过 HTML meta 标签，或者 设定 IIS 服务器 来声明 P3P。

### 浏览器对设置第三方cookie的态度
浏览器可不一定买广告商的账，能提供更好的隐私保障，才能更加讨好用户。

 - IE

默认不接受第三方 Cookie，不过采用 P3P 协议即可。（理论上讲，服务商应该在协议里真实地反映网站的信息搜集行为，若声明的隐私政策与实际行为不符，是要负法律责任的。）

 - Chrome、Firefox

默认接受。但用户可以选择不接受。

 - Safari中默认会阻止第三方cookie。


目前想在较新版本的 Safari 上种植第三方 Cookie，必须 让这个域在顶层窗口打开 （内嵌 iframe 是无效的）。也就是说，如果要种第三方 cookie，要么将页面先跳到这个域，写入cookie，再跳回来；要么弹出一个新窗口，写入cookie，再关闭弹窗。

不过幸好， Safari 认为相同父域名的站点不属于第三方，比如我们可以在 1.qq.com 里轻易地种下 2.qq.com 的 Cookie，想种 1.url.cn 则不行。
