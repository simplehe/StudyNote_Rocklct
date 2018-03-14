## data-*属性
在HTML5中添加了data-*的方式来自定义属性，所谓data-*实际上上就是data-前缀加上自定义的属性名，使用这样的结构可以进行数据存放。使用data-*可以解决自定义属性混乱无管理的现状。

使用 data-* 属性来嵌入自定义数据：

data-*有两种设置方式，可以直接在HTML元素标签上书写

其中的data-age就是一种自定义属性，当然我们也可以通过JavaScript来对其进行操作，HTML5中元素都会有一个dataset的属性，这是一个DOMStringMap类型的键值对集合

``` javascript
var test = document.getElementById('test');
        test.dataset.my = 'Byron';
```

如果我们设置data-*后面的名字带有连字符"-"，则
 - css中使用带连字符的形式
 - js中把连字符改成驼峰命名法才能调用

比如
``` html
<div id="test" data-birth-date = "19950808"></div>
```

则css中写属性值
```
[data-birth-date]{
   ......
}
```

js获取
``` javascript
getElementByID("test").dataset.birthDate = .....;

```


总之想要读取的时候，就用dataset这个属性去读取。
