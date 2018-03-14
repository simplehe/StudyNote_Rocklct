## flexbox布局


阮一峰的日志
<a>http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html</a>

### 用法
指定某容器采用flex布局,webkit内核浏览器要添加前缀

``` css
.box{
  display: -webkit-flex; /* Safari */
  display: flex;
}

```

设置为flex布局的容器，他的子元素自动成为容器元素flex item。

容器设置两条轴，一条主轴，一条交叉轴。默认主轴为水平轴。