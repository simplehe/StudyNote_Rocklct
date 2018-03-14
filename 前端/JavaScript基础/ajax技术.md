## ajax技术
AJAX，Asynchronous JavaScript and XML (异步的JavaScript和XML)，一种创建交互式网页应用的网页开发技术方案。

是在不需要重新加载网页的情况下，与服务器进行数据交互并更新部分网页的技术。

*Ajax是不能跨域的，如需跨域，可以使用document.domain=‘a.com’；或者使用服务器代理，代理XMLHttpRequest文件*

Ajax的核心是创建 XMLHttpRequest 对象

所有现代浏览器（IE7+、Firefox、Chrome、Safari 以及 Opera）均内建 XMLHttpRequest 对象。

### 过程

#### 1.创建Ajax核心对象XMLHttpRequest
``` javascript
var xmlhttp;  
if (window.XMLHttpRequest)  
  {// 兼容 IE7+, Firefox, Chrome, Opera, Safari  
  xmlhttp=new XMLHttpRequest();  
  }  
else  
  {// 兼容 IE6, IE5  
  xmlhttp=newActiveXObject("Microsoft.XMLHTTP");  
  }  
```

#### 2连接服务器
``` javascript
xmlhttp.open(method,url,async);  

// example
xmlhttp.open("POST","ajax_test.html",true);  
xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");  
```

 - method：请求的类型；GET 或 POST
 - url：文件在服务器上的位置
 - async：true（异步）或 false（同步）

#### 设置回调函数解释获得的数据

``` javascript
xhr.onreadystatechange = function() {
    if (xhr.readyState == 4) {
        if (xhr.status == 200) {
            // 处理数据
            // alert(xhr.responseText);
            oB.innerHTML = xhr.responseText;
            /*var str = xhr.responseText;
            oB.innerHTMl = eval(str);*/
        } else {
            alert('出错了,Erro' + xhr.status);
        }
    }
```

一共有5中请求状态，从0 到 4 发生变化。

0: 请求未初始化

1: 服务器连接已建立

2: 请求已接收

3: 请求处理中

4: 请求已完成，且响应已就绪

