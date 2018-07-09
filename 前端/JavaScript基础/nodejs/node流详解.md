## node流详解
node处理数据，有buffer模式和stream模式。buffer模式就是先读到内存再操作，显然数据大的时候就不行了，使用流模式是很有必要的。

在之前的buffer和stream笔记里已经介绍了两种对象

nodejs里面的stream一般分四种, 其中转换流是一种特殊的读写流：

 - 输入流(stream.Readable)
 - 输出流(stream.Writable)
 - 读写流(stream.Duplex)
 - 转换流(stream.Transform)

 另外, nodejs里面的流有两种模式, 二进制模式和对象模式.

 - 二进制模式, 每个分块都是buffer或者string对象.
 - 对象模式, 流内部处理的是一系列普通对象.

### 输入流(stream.Readable)
输入流的使用方法 **有两种**。** 一个是非流动模式, 一个是流动模式**.
非流动模式就是直接调用read()方法, 被动模式就是监听data事件.

#### 非流动模式(调用read方法)
``` js
// 非流动模式
process.stdin
    .on('readable', function() {
        // 有数据到了, 拼命读, 直到读完.
        var chunk;
        console.log('New data available');
        while((chunk = process.stdin.read()) !== null) {
            console.log(
            'Chunk read: (' + chunk.length + ') "' + chunk.toString() + '"'
            );
        }})
    .on('end', function() {
       process.stdout.write('End of stream');
    });

```

#### 流动模式(处理data事件的回调函数)
``` js
// 流动模式
process.stdin
     .on('data', function(chunk) {
       console.log('New data available');
       console.log(
         'Chunk read: (' + chunk.length + ')" ' +
         chunk.toString() + '"'
       );
     })
     .on('end', function() {
       process.stdout.write('End of stream');
     });


```

#### 实现输入流
实现一个输入流也很简单, 主要是

 - 继承Readable类
 - 实现_read(size)接口(一般带下划线的表示内部函数, 调用者不要直接调用, 相当于C++里面的protect方法, 只是javascript里面没有对方法做区分, 只能是命名上面区分一下了).

``` js
 // randomStream.js
var stream = require('stream');
var util = require('util');
var chance = require('chance').Chance();

function RandomStream(options) {
    // option支持3个参数
    // encoding String 用于转换Buffer到String的编码类型(默认null)
    // objMode Boolean 用户指定是否是对象模式(默认false)
    // highWaterMark Number 最高水位(可读的最大数据量), 默认是16K
    stream.Readable.call(this, options);
}
util.inherits(RandomStream, stream.Readable);
RandomStream.prototype._read = function(size) {
    // 这是一个随机产生数据的流, 5%的概率输出null, 也就是流停止.
    var chunk = chance.string();
    console.log('Pushing chunk of size:' + chunk.length);
    this.push(chunk, 'utf8');
    if(chance.bool({likelihood: 5})) {
       this.push(null);
    }
}
module.exports = RandomStream;
```

在一个可读流里，我们可以持续随机地往流里push数据。push null即为结束可读流的数据读取。
