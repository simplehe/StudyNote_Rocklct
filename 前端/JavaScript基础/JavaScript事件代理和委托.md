## JavaScript事件代理和委托

出自 https://segmentfault.com/a/1190000002613617

在JavaScript中，经常会碰到要监听列表中多项li的情形，假设我们有一个列表如下：


``` html
<ul id="list">
  <li id="item1">item1</li>
  <li id="item2">item2</li>
  <li id="item3">item3</li>
  <li id="item4">item4</li>
</ul>
```

如果我们要实现以下功能：当鼠标点击某一li时，alert输出该li的内容，我们通常的写法是这样的：

当列表项比较少时，直接给每个li添加onclick事件
列表项比较多时，在onload时就给每个列表项调用监听
第一种方法比较简单直接，但是没有顾及到html与JavaScript的分离，不建议使用，第二种方法的代码如下：

``` javascript
window.onload=function(){
  var ulNode=document.getElementById("list");
  var liNodes=ulNode.childNodes||ulNode.children;
  for(var i=0;i<liNodes.length;i++){
    liNodes[i].addEventListener('click',function(e){
      alert(e.target.innerHTML);
    },false);
  }
}
```

由上可以看出来，假如不停的删除或添加li，则function（）也要不停的更改操作，易出错，因此推荐使用事件代理，在使用事件代理之前，我们先来了解一下事件阶段(event phase)：

事件阶段：
当一个DOM事件被触发的时候，他并不是只在它的起源对象上触发一次，而是会经历三个不同的阶段。简而言之：事件一开始从文档的根节点流向目标对象(捕获阶段)，然后在目标对向上被触发(目标阶段)，之后再回溯到文档的根节点(冒泡阶段)如图所示（图片来自W3C）：



事件捕获阶段(Capture Phase)

事件的第一个阶段是捕获阶段。事件从文档的根节点出发，随着DOM树的结构向事件的目标节点流去。途中经过各个层次的DOM节点，并在各节点上触发捕获事件，直到到达时间的目标节点。捕获阶段的主要任务是简历传播路径，在冒泡阶段，时间会通过这个路径回溯到文档根节点。


element.removeEventListener(&ltevent-name>, <callback>, <use-capture>);

我们通过上面的这个函数来给节点设置监听，可以通过将;设置成true来为事件的捕获阶段添加监听回调函数。在实际应用中，我们并没有太多使用捕获阶段监听的用例，但是通过在捕获阶段对事件的处理，我们可以阻止类似click事件在某个特定元素上被触发。

``` javascript
var form=document.querySeletor('form');
form.addEventListener('click',function(e){
  e.stopPropagation();
  },true);

```
如果你对这种用法不是很了解的话，最好还是将设置为false或者undefined，从而在冒泡阶段对事件进行监听。

目标阶段(Target Phase)

当事件到达目标节点时，事件就进入了目标阶段。事件在目标节点上被触发，然后逆向回流，知道传播到最外层的文档节点。

对于多层嵌套的节点，鼠标和指针事件经常会被定位到最里层的元素上。假设，你在一个div元素上设置了click的监听函数，而用户点击在了这个div元素内部的p元素上，那么p元素就是这个时间的目标元素。事件冒泡让我们可以在这个div或者更上层的元素上监听click事件，并且时间传播过程中触发回调函数。

冒泡阶段(Bubble Phase)

事件在目标事件上触发后，并不在这个元素上终止。它会随着DOM树一层层向上冒泡，直到到达最外层的根节点。也就是说，同一事件会一次在目标节点的父节点，父节点的父节点...直到最外层的节点上触发。

绝大多数事件是会冒泡的，但并非所有的。具体可见：规范说明

由上我们可以想到，可以使用事件代理来实现对每一个li的监听。代码如下：

``` javascript
window.onload=function(){
  var ulNode=document.getElementById("list");
  ulNode.addEventListener('click',function(e){
       if(e.target&&e.target.nodeName.toUpperCase()=="LI"){/*判断目标事件是否为li*/
         alert(e.target.innerHTML);
       }
     },false);
  
};
```