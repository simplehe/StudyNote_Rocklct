#### position的值， relative和absolute分别是相对于谁进行定位的？


- `absolute` :生成绝对定位的元素， 相对于最近一级的 定位不是 static 的父元素来进行定位。

- `fixed` （老IE不支持）生成绝对定位的元素，通常相对于浏览器窗口或 frame 进行定位。

- `relative` 生成相对定位的元素，相对于其在普通流中的位置进行定位。

- `static`  默认值。没有定位，元素出现在正常的流中

- `sticky` 生成粘性定位的元素，容器的位置根据正常文档流计算得出

<br>


#### 渐进增强和优雅降级

渐进增强 ：针对低版本浏览器进行构建页面，保证最基本的功能，然后再针对高级浏览器进行效果、交互等改进和追加功能达到更好的用户体验。



优雅降级 ：一开始就构建完整的功能，然后再针对低版本浏览器进行兼容。

>CSS中` link` 和`@import `的区别是？

    (1) link属于HTML标签，而@import是CSS提供的;

    (2) 页面被加载的时，link会同时被加载，而@import被引用的CSS会等到引用它的CSS文件被加载完再加载;

    (3) import只在IE5以上才能识别，而link是HTML标签，无兼容问题;

    (4) link方式的样式的权重 高于@import的权重.

>介绍一下box-sizing属性？


`box-sizing`属性主要用来控制元素的盒模型的解析模式。默认值是`content-box`。


- `content-box`：让元素维持W3C的标准盒模型。元素的宽度/高度由`border + padding + content`的宽度/高度决定，设置`width/height`属性指的是`content`部分的宽/高

- `border-box`：让元素维持IE传统盒模型（IE6以下版本和IE6~7的怪异模式）。设置`width/height`属性指的是`border + padding + content`
