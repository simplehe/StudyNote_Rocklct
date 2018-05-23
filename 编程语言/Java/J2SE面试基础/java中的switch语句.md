## switch语句
需要注意的几个地方

``` java
switch（A）{
case B;
}
```

**A部分必须是int型的，或者是能够自动进行饮试转换成int型的表达式。也就是说A部分可以是byte/short/char/int型的，还有Integer。**

case后的值可以是常数值或final型的变量，当然也得是可以转换为int类型的。

java1.7以后，case语句后面也许是可以添加变量了。

还要注意，如果不写break，将会顺序执行直至break或者default。
