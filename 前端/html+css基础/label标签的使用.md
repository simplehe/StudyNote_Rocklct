## css label标签
<label> 标签为 input 元素定义标注（标记）。
label 元素不会向用户呈现任何特殊效果。不过，它为鼠标用户改进了可用性。如果您在 label 元素内点击文本，就会触发此控件。就是说，当用户选择该标签时，浏览器就会自动将焦点转到和标签相关的表单控件上。

也即是说**label标签主要用于绑定一个表单元素, 当点击label标签的时候, 被绑定的表单元素就会获得输入焦点**

绑定的方法是：将for属性值指定为目的控件（绑定控件）的ID。一般情况下，在使用单选框和复选框时用label绑定比较常见。

例如
``` html
<label for="male">Male</label>
<input type="radio" name="sex" id="male" />
```

亦或者显示定义
``` html
<label>Click me <input type="text" id="User" name="Name" /></label>
```