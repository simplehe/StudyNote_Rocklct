## IDA Pro
交互式反汇编器专业版（Interactive Disassembler Professional），人们常称其为IDA Pro，或简称为IDA。是目前最棒的一个**静态反编译软件**，为众多0day世界的成员和ShellCode安全分析人士不可缺少的利器！

注意它是用作静态反编译分析的。

IDA pro经常用来找出程序的入口点，也就是在function标签下的，start函数的地址。

### 使用技巧
使用IDA做静态的反汇编工作时，首先们必须注意几点

- 不要太关注汇编代码的细节，而要**关注function call和strings**。从function call中我们能知道exe程序调用了哪些系统函数，或者说调用了哪些自定义函数。

 - 不要深入的研究所有函数，在function标签页中，按大小排序，一般而言，**size比较大的function尤为重要**。

 - 时刻用chart refers来比对，有想看的函数，按G件，直接goto地址

#### 右键chart refers to/from
对着黄色highlight的函数名右键，选择chart refers to。可以看到这个函数的，被调用情况。同理，refers from,可以看到这个函数，调用其他函数的情况。
