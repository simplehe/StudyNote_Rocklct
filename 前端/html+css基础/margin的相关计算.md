## margin知识点

margin有很多知识点必须要注意。

首先要注意一下两个点

 - margin的值时百分比时，是根据父元素内容盒**宽度**来计算的。注意top和bottom也是根据内容盒的**宽度**来计算，并非父元素高度！！
 - margin-top和bottom对行内元素(行内非替换元素)是无效的，比如i,span;

### 垂直margin相邻

垂直方向上满足一定条件，即可说两个margin是垂直毗连的。以下四种情况满足其一即可：

 1. 父元素top margin和第一个元素的top margin
 2. 父元素的bottom和最后一个元素的bottom
 3. 元素的bottom和这个元素相邻兄弟元素的top
 4. 一个元素，没有BFC，没有正常流子元素，min-height和height都是0，那它的上下两个margin也是毗连的。

除了**毗连**这个概念外，还有**相邻**这个概念。满足毗连条件还不够，一下定义相邻

 1. 两个margin是垂直毗连的。
 2. 两个margin都是正常流块级元素，并且在同一个BFC中。
 3. 两个margin之间没有行盒(line box),清浮动后的空隙，padding和border都没有。

以上三个条件全部满足，就是相邻了。

满足垂直相邻的元素，margin会被**折叠**(当做同一个margin)，折叠的时候取二者正数最大值，正负则累加，负数时则取最小值。

具体来说经常遇到的是div不设padding和border，第一个子元素的top margin经常会和这个父元素自身的margin合并。

但是也要注意，根元素(html元素)不被折叠；如果一个元素，它的 top margin 和 bottom margin 是相邻的，并且有清除浮动后的空隙（clearance），这个元素的 margin 可以跟兄弟元素的 margin 折叠，但是折叠后的 margin 不能跟父元素的 bottom margin 折叠。