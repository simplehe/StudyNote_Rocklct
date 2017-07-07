## For与Foreach
java有两种循环格式for和foreach

foreach 循环语法：
``` java
for (int i = 0; i < integers.length; i++) {
    System.out.println(intergers[i]);
}
```

foreach语句是**java5**的新特征之一，在遍历数组、集合方面，foreach为开发人员提供了极大的方便。

foreach语句是for语句的特殊简化版本，但是foreach语句并不能完全取代for语句，然而，任何的foreach语句都可以改写为for语句版本。

foreach并不是一个关键字，习惯上**将这种特殊的for语句格式称之为“foreach”语句**。从英文字面意思理解foreach也就是“for 每一个”的意思。实际上也就是这个意思。

foreach实际上是语法糖，遍历数组时，他自动获取length然后索引。遍历Collection的时候，它获取的是迭代器。所以有实现迭代器的，都可以使用这个foreach的方法。

### 总结
foreach语句是for语句特殊情况下的增强版本，简化了编程，提高了代码的可读性和安全性（不用怕数组越界）。相对老的for语句来说是个很好的补充。提倡能用foreach的地方就不要再用for了。在用到对集合或者数组索引的情况下，foreach显得力不从心，这个时候是用for语句的时候了。
