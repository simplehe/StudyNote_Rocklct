## 公告牌demo
使用Thrift，首先安装编译Thrift

安装之前先安装boost，一个c++准标准库，

```
sudo apt-get install libboost-dev
```

然后下载Thirft，进到目录，

```
./configure
sudo make install
```

### 编写Thrift文件
新开个文件，编写以下内容

```
struct Message
{
	1: string id,
	2: string content,
}

service MessageService
{
	Message latestMessage(),
}
```

我们接下来就可以用Thrift框架把他转换成cpp代码。

运行

``` cmd
thrift -r --gen cpp message.thrift
```

目录下会产生一个cpp文件夹，里面是生成的源码
