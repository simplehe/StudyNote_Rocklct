## box-sizing

人们慢慢的意识到传统的盒子模型不直接，所以他们新增了一个叫做 box-sizing 的CSS属性。当你设置一个元素为 box-sizing: border-box; 时，此元素的内边距和边框不再会增加它的宽度。

因此为使盒子不要被padding啊，边框给撑开使得宽度不一致，会把box-sizing设置成border-box

一些CSS开发者想要页面上所有的元素都有如此表现。所以开发者们把以下CSS代码放在他们页面上：
``` css
* {
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
}
```

这样可以确保所有的元素都会用这种更直观的方式排版。

不过 box-sizing 是个很新的属性，目前你还应该像我上面例子中那样使用 -webkit- 和 -moz- 前缀。这可以启用特定浏览器实验中的特性。同时记住它是支持IE8+的。