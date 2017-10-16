## OllyDbg
OLLYDBG是一个新的**动态追踪工具**，将IDA与SoftICE结合起来的思想，Ring 3级调试器，非常容易上手，己代替SoftICE成为当今最为流行的调试解密工具了。同时还支持插件扩展功能，是目前最强大的调试工具。

### 启动
我们直接在OllyDbg启动要分析的exe文件，注意一下入口点，OllyDbg启动的exe不保证能找对真正的入口点，所以需要使用工具，比如IDA Pro去找到真正的Strat点，然后再OllyDbg双击那个位置(用crtl g来跳转)来方便启动。

使用OllyDbg时，我们要面向String去解决问题。用右键Search for->all refered text string，来找那些我们在静态分析时找出的可疑变量。
