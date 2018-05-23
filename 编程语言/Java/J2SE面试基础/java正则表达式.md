## java正则表达式

正则表达式可以用来搜索、编辑或处理文本。正则表达式并不仅限于某一种语言，但是在每种语言中有细微的差别。

Java正则表达式的类在 java.util.regex 包中，包括三个类：Pattern,Matcher 和 PatternSyntaxException。

 - Pattern对象是正则表达式的已编译版本。他没有任何公共构造器，我们通过传递一个正则表达式参数给公共静态方法 compile 来创建一个pattern对象。

 - Matcher是用来匹配输入字符串和创建的 pattern 对象的正则引擎对象。这个类没有任何公共构造器，我们用patten对象的matcher方法，使用输入字符串作为参数来获得一个Matcher对象。然后使用matches方法，通过返回的布尔值判断输入字符串是否与正则匹配。

 - 如果正则表达式语法不正确将抛出PatternSyntaxException异常。


### 捕获组
捕获组是一种将多个字符抽象为一个处理单元的方法。他们通过用括号将字符分组来创建。

捕获组通过从左向右计算**匹配左括号**来进行计数。在正则表达式((A)(B(C)))中，这里有四个组：

 - ((A)(B(C)))

 - (A)

 - (B\(C\))

 - \(C\)


为了在表达式中计算有多少个组，可以调用 matcher 对象中的 groupCount 方法。 groupCount 方法返回一个 int 类型来显示正则表达式中的捕获组的数量。

因为java正则表达式中很多诸如 \n这种形式，所以经常需要加转义字符让他正确转为字符串。

### 正则表达式贪婪模式

**贪婪匹配:正则表达式一般趋向于最大长度匹配，也就是所谓的贪婪匹配。**


java默认是贪婪模式；在量词后面直接加上一个问号？就是非贪婪模式。

量词：
 - {m,n}：m到n个
 - *：任意多个
 - +：一个到多个


### 语法表格

<table>
<thead>
<tr>
<th style="text-align: left;">子表达式</th>
<th style="text-align: left;">匹配对应</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: left;">^</td>
<td style="text-align: left;">匹配一行的开头</td>
</tr>
<tr>
<td style="text-align: left;">$</td>
<td style="text-align: left;">匹配一行的结尾</td>
</tr>
<tr>
<td style="text-align: left;">.</td>
<td style="text-align: left;">匹配除了换行符的任何单个字符，也可以利用 m 选项允许它匹配换行符</td>
</tr>
<tr>
<td style="text-align: left;">[...]</td>
<td style="text-align: left;">匹配括号内的任意单个字符。</td>
</tr>
<tr>
<td style="text-align: left;">[^...]</td>
<td style="text-align: left;">匹配不在括号内的任意单个字符。</td>
</tr>
<tr>
<td style="text-align: left;">\A</td>
<td style="text-align: left;">整个字符串的开始</td>
</tr>
<tr>
<td style="text-align: left;">\z</td>
<td style="text-align: left;">整个字符串的结束</td>
</tr>
<tr>
<td style="text-align: left;">\Z</td>
<td style="text-align: left;">整个字符串的结束，除了最后一行的结束符</td>
</tr>
<tr>
<td style="text-align: left;">re*</td>
<td style="text-align: left;">匹配0或者更多的前表达事件</td>
</tr>
<tr>
<td style="text-align: left;">re+</td>
<td style="text-align: left;">匹配1个或更多的之前的事件</td>
</tr>
<tr>
<td style="text-align: left;">re?</td>
<td style="text-align: left;">匹配0或者1件前表达事件</td>
</tr>
<tr>
<td style="text-align: left;">re{ n}</td>
<td style="text-align: left;">匹配特定的n个前表达事件</td>
</tr>
<tr>
<td style="text-align: left;">re{ n,}</td>
<td style="text-align: left;">匹配n或者更多的前表达事件</td>
</tr>
<tr>
<td style="text-align: left;">re{ n, m}</td>
<td style="text-align: left;">匹配至少n最多m件前表达事件</td>
</tr>
<tr>
<td style="text-align: left;">a| b</td>
<td style="text-align: left;">匹配a或者b</td>
</tr>
<tr>
<td style="text-align: left;">(re)</td>
<td style="text-align: left;">正则表达式组匹配文本记忆</td>
</tr>
<tr>
<td style="text-align: left;">(?: re)</td>
<td style="text-align: left;">没有匹配文本记忆的正则表达式组</td>
</tr>
<tr>
<td style="text-align: left;">(?&gt; re)</td>
<td style="text-align: left;">匹配无回溯的独立的模式</td>
</tr>
<tr>
<td style="text-align: left;">\w</td>
<td style="text-align: left;">匹配单词字符</td>
</tr>
<tr>
<td style="text-align: left;">\W</td>
<td style="text-align: left;">匹配非单词字符</td>
</tr>
<tr>
<td style="text-align: left;">\s</td>
<td style="text-align: left;">匹配空格。等价于 [\t\n\r\f]</td>
</tr>
<tr>
<td style="text-align: left;">\S</td>
<td style="text-align: left;">匹配非空格</td>
</tr>
<tr>
<td style="text-align: left;">\d</td>
<td style="text-align: left;">匹配数字. 等价于 [0-9]</td>
</tr>
<tr>
<td style="text-align: left;">\D</td>
<td style="text-align: left;">匹配非数字</td>
</tr>
<tr>
<td style="text-align: left;">\A</td>
<td style="text-align: left;">匹配字符串的开始</td>
</tr>
<tr>
<td style="text-align: left;">\Z</td>
<td style="text-align: left;">匹配字符串的末尾，如果存在新的一行，则匹配新的一行之前</td>
</tr>
<tr>
<td style="text-align: left;">\z</td>
<td style="text-align: left;">匹配字符串的末尾</td>
</tr>
<tr>
<td style="text-align: left;">\G</td>
<td style="text-align: left;">匹配上一次匹配结束的地方</td>
</tr>
<tr>
<td style="text-align: left;">\n</td>
<td style="text-align: left;">返回参考捕获组号“N”</td>
</tr>
<tr>
<td style="text-align: left;">\b</td>
<td style="text-align: left;">不在括号里时匹配单词边界。在括号里时匹配退格键</td>
</tr>
<tr>
<td style="text-align: left;">\B</td>
<td style="text-align: left;">匹配非词边界</td>
</tr>
<tr>
<td style="text-align: left;">\n, \t, etc.</td>
<td style="text-align: left;">匹配换行符，回车符，制表符，等</td>
</tr>
<tr>
<td style="text-align: left;">\Q</td>
<td style="text-align: left;">引用字符的初始，结束于\E</td>
</tr>
<tr>
<td style="text-align: left;">\E</td>
<td style="text-align: left;">结束由\Q开始的引用</td>
</tr>
</tbody>
</table>
