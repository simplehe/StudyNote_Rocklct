## yara
**YARA是一款旨在帮助恶意软件研究人员识别和分类恶意软件样本的开源工具**（由virustotal的软件工程师Victor M. Alvarezk开发），使用YARA可以基于文本或二进制模式创建恶意软件家族描述信息，当然也可以是其他匹配信息。

YARA的每一条描述或规则都由一系列字符串和一个布尔型表达式构成，并阐述其逻辑。YARA规则(yara rule)可以提交给文件或在运行进程，以帮助研究人员识别其是否属于某个已进行规则描述的恶意软件家族。比如下面这个例子：

```
rule silent_banker : banker
{
    meta:
        description = "This is just an example"
        thread_level = 3
        in_the_wild = true
    strings:
        $a = {6A 40 68 00 30 00 00 6A 14 8D 91}
        $b = {8D 4D B0 2B C1 83 C0 27 99 6A 4E 59 F7 F9}
        $c = "UVODFRYSIHLNWPEJXQZAKCBGMT"
    condition:
        $a or $b or $c
}
```

以上规则告诉YARA任何包含有$a、$b、$c字符串的文件都被标识为slient_banker。这仅仅是一个简单的例子，YARA的规则可以复杂和强大到支持通配符、大小写敏感字符串、正则表达式、特殊符号以及其他特性。

zwlj:我们使用yara(基于一种规则来检测malware的程序)来帮助我们从系统中根据一些特定特征检测出一系列同类malware。比如说，我们手里发现了一个malware，我们希望知道系统里还有没有其他的别的类似的malware，我们因此就需要对手里这个malware进行分析，抽取出一些非常unique的具有特征的信息，比如malware author，或者hash，或者一些重要string。我们根据这些特征编写yara rule，从而运行yara找出一些类似特征的malware。
