---
tags: nodejs
date:  2018-07-20 18:29
title:   nodejs的stream
categories: nodejs
---



[toc]
#### nodejs的stream
##### 一.  为什么需要流（Stream）？

　　举个例子，如果要读取一个文件，一次性读取需要占用大内存，是不可取的。因此就有了流，用流会很方便，可以帮我们避免这样的问题，调用其接口不用关心底层如何实现。

##### 二. 什么是流（Stream）？

　　流（Stream）是可读，可写或双工的。可以通过require('stream')加载流的基类，其中包括四类流， Readable 流、Writable 流、Duplex 流和Transform 流的基类。

　　另外如果觉得上述四类基类流不能满足需求，可以编写自己的扩充类流。像我们Team现在正做的Node项目，就重写了Transform类以供使用。

按照官方的API文档，步骤如下：

- 在您的子类中扩充适合的父类。（例如util.inherits(MyTransform, Transform); ）
- 在您的构造函数中调用父类的构造函数，以确保内部的机制被正确初始化。
- 实现一个或多个特定的方法，参见下面的细节。　

在nodejs中，有四种stream类型：



<!-- more -->


- Readable：用来读取数据，比如 fs.createReadStream()。
- Writable：用来写数据，比如 fs.createWriteStream()。
- Duplex：可读+可写，比如 net.Socket()。
- Transform：在读写的过程中，可以对数据进行修改，比如 zlib.createDeflate()（数据压缩/解压）。

##### 三. Readable流（可读流）介绍

　  Readable（可读）流接口是对您正在读取的数据的来源的抽象。换言之，数据出自一个可读流。

　  Readable 流有两种“模式”：流动模式和暂停模式。

　  当处于流动模式时，数据由底层系统读出，并尽可能快地提供给您的程序；当处于暂停模式时，您必须明确地调用 stream.read() 来取出若干数据块。流默认处于暂停模式。

　  

　  A. 通过以下三种方法，可读流会被切换到流动模式

　  　 1. 添加一个'data'事件处理器来监听数据。

　  　 2. 调用 resume()方法来明确开启数据流。

    　 3. 调用 pipe()方法将数据发送到一个可写流（Writable）。

　  　 之前我一直对pipe()方法有疑问，不清楚其用法。现在了解，当我们用pipe()为可读流指定了一个接受者（可写流）的时候，数据才会真正的被从底层系统读出，传递给可写流。

　  B. 下面介绍Readable流有以下几种事件

　　   1. 'Readable'事件

　　   2. 'data'事件 - 数据正在传递时，触发该事件（以chunk数据块为对象）

　　   3. 'end'事件 - 数据传递完成后，会触发该事件。

　　   4. 'close'事件

　　   5. 'error'事件

　　   所有这些事件都可以在官方API文档中找到例子。

 

　   C. 下面介绍Readable流很重要的一个方法，pipe()方法。

　　   该方法从可读流中拉取所有数据，并写入到所提供的目标（可写流）。该方法能自动控制流量以避免目标被快速读取的可读流所淹没。

　　   值得注意的是，默认情况下，当数据传送完毕，触发'end'事件时，会同时触发目标（可写流）的'end'事件，导致目标不再可写。

###### 　　　举个简单的小例子

```
1 //http.js
 2 
 3 var http = require('http');
 4 var fs = require('fs');
 5 
 6 http.createServer(function(req, res){
 7     var stream = fs.createReadStream(__dirname + '/data.txt');
 8     stream.pipe(res);
 9 }).listen(3000);
10 
11 console.log('now we are listening 3000 port');
```
   data.txt文件内容如下：
   ![image](http://ww1.sinaimg.cn/mw690/773c8d3dgw1ehi37a16dvj208u03rdft.jpg)
  
  当执行此段代码后，用户访问http://127.0.0.1:3000/，会得到如下响应：
  ![image](http://ww3.sinaimg.cn/mw690/773c8d3dgw1ehi37a9k5zj209m02qdfq.jpg)
  
  此时，创建此Server后，用户访问请求过来，Server会创建一个可读流，当调用stream.pipe(res)为可读流指定目标后，可读流stream会开始从文件data.txt中读取数据，数据写入res（可写流）完毕后，自动调用res的end()方法，结束响应，可写流不再写入。
  
  第二个例子
  
  以下都是nodejs中常见的Readable Stream，当然还有其他的，可自行查看文档。

- http.IncomingRequest
- fs.createReadStream()
- process.stdin
- 其他

**例子一：**


```
var fs = require('fs');

fs.readFile('./sample.txt', 'utf8', function(err, content){
    // 文件读取完成，文件内容是 [你好，我是程序猿小卡]
    console.log('文件读取完成，文件内容是 [%s]', content);
});
```

**例子二：**


```
var fs = require('fs');

var readStream = fs.createReadStream('./sample.txt');
var content = '';

readStream.setEncoding('utf8');

readStream.on('data', function(chunk){
    content += chunk;
});

readStream.on('end', function(chunk){
    // 文件读取完成，文件内容是 [你好，我是程序猿小卡]
    console.log('文件读取完成，文件内容是 [%s]', content);
});
```

**例子三：**

这里使用了.pipe(dest)，好处在于，如果文件


```
var fs = require('fs');

fs.createReadStream('./sample.txt').pipe(process.stdout);
```

注意：这里只是原封不动的将内容输出到控制台，所以实际上跟前两个例子有细微差异。可以稍做修改，达到上面同样的效果


```
var fs = require('fs');

var onEnd = function(){
    process.stdout.write(']');    
};

var fileStream = fs.createReadStream('./sample.txt');
fileStream.on('end', onEnd)

fileStream.pipe(process.stdout);

process.stdout.write('文件读取完成，文件内容是[');

// 文件读取完成，文件内容是[你好，我是程序猿小卡]
```
##### 四. Writable流（可写流）介绍
Writable（可写）流接口是对写入数据的目标的抽象。

　  可写流重要的两个方法，

　  1. write()方法

　　   该方法向底层系统写入数据，并在数据被处理完毕后调用所给的回调。

　  2. end()方法

　　　当不再写入数据时，调用该方法，停止写入。在调用end()后，再调用write()方法会产生错误。

同样以写文件为例子，比如想将hello world写到sample.txt里。

**例子一**：


```
var fs = require('fs');
var content = 'hello world';
var filepath = './sample.txt';

fs.writeFile(filepath, content);
```

**例子二：**


```
var fs = require('fs');
var content = 'hello world';
var filepath = './sample.txt';

var writeStram = fs.createWriteStream(filepath);
writeStram.write(content);
writeStram.end();
```
五、Duplex Stream

最常见的Duplex stream应该就是net.Socket实例了，在前面的文章里有接触过，这里就直接上代码了，这里包含服务端代码、客户端代码。

服务端代码：


```
var net = require('net');
var opt = {
    host: '127.0.0.1',
    port: '3000'
};

var client = net.connect(opt, function(){
    client.write('msg from client');  // 可写
});

// 可读
client.on('data', function(data){
    // server: msg from client [msg from client]
    console.log('client: got reply from server [%s]', data);
    client.end();
});
```

客户端代码：


```
var net = require('net');
var opt = {
    host: '127.0.0.1',
    port: '3000'
};

var client = net.connect(opt, function(){
    client.write('msg from client');  // 可写
});

// 可读
client.on('data', function(data){
    // lient: got reply from server [reply from server]
    console.log('client: got reply from server [%s]', data);
    client.end();
});
```

六、Transform Stream
Transform stream是Duplex stream的特例，也就是说，Transform stream也同时可读可写。跟Duplex stream的区别点在于，Transform stream的输出与输入是存在相关性的。

常见的Transform stream包括zlib、crypto，这里举个简单例子：文件的gzip压缩。


```
var fs = require('fs');
var zlib = require('zlib');

var gzip = zlib.createGzip();

var inFile = fs.createReadStream('./extra/fileForCompress.txt');
var out = fs.createWriteStream('./extra/fileForCompress.txt.gz');

inFile.pipe(gzip).pipe(out);
```
#### Node.js Buffer(缓冲区)
JavaScript 语言自身只有字符串数据类型，没有二进制数据类型。
但在处理像TCP流或文件流时，必须使用到二进制数据。因此在 Node.js中，定义了一个 Buffer 类，该类用来创建一个专门存放二进制数据的缓存区。

*在v6.0之前创建Buffer对象直接使用new Buffer()构造函数来创建对象实例，但是Buffer对内存的权限操作相比很大，可以直接捕获一些敏感信息，所以在v6.0以后，官方文档里面建议使用 Buffer.from() 接口去创建Buffer对象。*

##### Buffer 与字符编码
Buffer 实例一般用于表示编码字符的序列，比如 UTF-8 、 UCS2 、 Base64 、或十六进制编码的数据。 通过使用显式的字符编码，就可以在 Buffer 实例与普通的 JavaScript 字符串之间进行相互转换。

```
const buf = Buffer.from('runoob', 'ascii');

// 输出 72756e6f6f62
console.log(buf.toString('hex'));

// 输出 cnVub29i
console.log(buf.toString('base64'));
```

Node.js 目前支持的字符编码包括：
- ascii - 仅支持 7 位 ASCII 数据。如果设置去掉高位的话，这种编码是非常快的。
- utf8 - 多字节编码的 Unicode 字符。许多网页和其他文档格式都使用 UTF-8 。
- utf16le - 2 或 4 个字节，小字节序编码的 Unicode 字符。支持代理对（U+10000 至 U+10FFFF）。
- ucs2 - utf16le 的别名。
- base64 - Base64 编码。
- latin1 - 一种把 Buffer 编码成一字节编码的字符串的方式。
- binary - latin1 的别名。
- hex - 将每个字节编码为两个十六进制字符。
##### 创建 Buffer 类
Buffer 提供了以下 API 来创建 Buffer 类：
- Buffer.alloc(size[, fill[, encoding]])： 返回一个指定大小的 Buffer 实例，如果没有设置 fill，则默认填满 0
- Buffer.allocUnsafe(size)： 返回一个指定大小的 Buffer 实例，但是它不会被初始化，所以它可能包含敏感的数据
- Buffer.allocUnsafeSlow(size)
- Buffer.from(array)： 返回一个被 array 的值初始化的新的 Buffer 实例（传入的 array 的元素只能是数字，不然就会自动被 0 覆盖）
- Buffer.from(arrayBuffer[, byteOffset[, length]])： 返回一个新建的与给定的 ArrayBuffer 共享同一内存的 Buffer。
- Buffer.from(buffer)： 复制传入的 Buffer 实例的数据，并返回一个新的 Buffer 实例
- Buffer.from(string[, encoding])： 返回一个被 string 的值初始化的新的 Buffer 实例

```
// 创建一个长度为 10、且用 0 填充的 Buffer。
const buf1 = Buffer.alloc(10);

// 创建一个长度为 10、且用 0x1 填充的 Buffer。 
const buf2 = Buffer.alloc(10, 1);

// 创建一个长度为 10、且未初始化的 Buffer。
// 这个方法比调用 Buffer.alloc() 更快，
// 但返回的 Buffer 实例可能包含旧数据，
// 因此需要使用 fill() 或 write() 重写。
const buf3 = Buffer.allocUnsafe(10);

// 创建一个包含 [0x1, 0x2, 0x3] 的 Buffer。
const buf4 = Buffer.from([1, 2, 3]);

// 创建一个包含 UTF-8 字节 [0x74, 0xc3, 0xa9, 0x73, 0x74] 的 Buffer。
const buf5 = Buffer.from('tést');

// 创建一个包含 Latin-1 字节 [0x74, 0xe9, 0x73, 0x74] 的 Buffer。
const buf6 = Buffer.from('tést', 'latin1');
```
##### 写入缓冲区
###### 语法
写入 Node 缓冲区的语法如下所示：

```
buf.write(string[, offset[, length]][, encoding])
```

###### 参数
参数描述如下：
- string - 写入缓冲区的字符串。
- offset - 缓冲区开始写入的索引值，默认为 0 。
- length - 写入的字节数，默认为 buffer.length
- encoding - 使用的编码。默认为 'utf8' 。
根据 encoding 的字符编码写入 string 到 buf 中的 offset 位置。 length 参数是写入的字节数。 如果 buf 没有足够的空间保存整个字符串，则只会写入 string 的一部分。 只部分解码的字符不会被写入。
###### 返回值
返回实际写入的大小。如果 buffer 空间不足， 则只会写入部分字符串。
###### 实例

```
buf = Buffer.alloc(256);
len = buf.write("www.runoob.com");

console.log("写入字节数 : "+  len);
```

执行以上代码，输出结果为：

```
$node main.js
写入字节数 : 14
```

##### 从缓冲区读取数据
###### 语法
读取 Node 缓冲区数据的语法如下所示：

```
buf.toString([encoding[, start[, end]]])
```

###### 参数
参数描述如下：
- encoding - 使用的编码。默认为 'utf8' 。
- start - 指定开始读取的索引位置，默认为 0。
- end - 结束位置，默认为缓冲区的末尾。
###### 返回值
解码缓冲区数据并使用指定的编码返回字符串。
###### 实例

```
buf = Buffer.alloc(26);
for (var i = 0 ; i < 26 ; i++) {
  buf[i] = i + 97;
}

console.log( buf.toString('ascii'));       // 输出: abcdefghijklmnopqrstuvwxyz
console.log( buf.toString('ascii',0,5));   // 输出: abcde
console.log( buf.toString('utf8',0,5));    // 输出: abcde
console.log( buf.toString(undefined,0,5)); // 使用 'utf8' 编码, 并输出: abcde
```

执行以上代码，输出结果为：

```
$ node main.js
abcdefghijklmnopqrstuvwxyz
abcde
abcde
abcde
```

##### 将 Buffer 转换为 JSON 对象
###### 语法
将 Node Buffer 转换为 JSON 对象的函数语法格式如下：

```
buf.toJSON()
```

当字符串化一个 Buffer 实例时，JSON.stringify() 会隐式地调用该 toJSON()。
###### 返回值
返回 JSON 对象。
###### 实例

```
const buf = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5]);
const json = JSON.stringify(buf);

// 输出: {"type":"Buffer","data":[1,2,3,4,5]}
console.log(json);

const copy = JSON.parse(json, (key, value) => {
  return value && value.type === 'Buffer' ?
    Buffer.from(value.data) :
    value;
});

// 输出: <Buffer 01 02 03 04 05>
console.log(copy);
```

执行以上代码，输出结果为：

```
{"type":"Buffer","data":[1,2,3,4,5]}
<Buffer 01 02 03 04 05>
```

##### 缓冲区合并
###### 语法
Node 缓冲区合并的语法如下所示：

```
Buffer.concat(list[, totalLength])
```
###### 参数
参数描述如下：
- list - 用于合并的 Buffer 对象数组列表。
- totalLength - 指定合并后Buffer对象的总长度。
###### 返回值
返回一个多个成员合并的新 Buffer 对象。
###### 实例

```
var buffer1 = Buffer.from(('菜鸟教程'));
var buffer2 = Buffer.from(('www.runoob.com'));
var buffer3 = Buffer.concat([buffer1,buffer2]);
console.log("buffer3 内容: " + buffer3.toString());
```

执行以上代码，输出结果为：

```
buffer3 内容: 菜鸟教程 www.runoob.com
```

##### 缓冲区比较
###### 语法
Node Buffer 比较的函数语法如下所示, 该方法在 Node.js v0.12.2 版本引入：

```
buf.compare(otherBuffer);
```

###### 参数
参数描述如下：
- otherBuffer - 与 buf 对象比较的另外一个 Buffer 对象。
###### 返回值
返回一个数字，表示 buf 在 otherBuffer 之前，之后或相同。
###### 实例

```
var buffer1 = Buffer.from('ABC');
var buffer2 = Buffer.from('ABCD');
var result = buffer1.compare(buffer2);

if(result < 0) {
   console.log(buffer1 + " 在 " + buffer2 + "之前");
}else if(result == 0){
   console.log(buffer1 + " 与 " + buffer2 + "相同");
}else {
   console.log(buffer1 + " 在 " + buffer2 + "之后");
}
```

执行以上代码，输出结果为：

```
ABC在ABCD之前
```

##### 拷贝缓冲区
###### 语法
Node 缓冲区拷贝语法如下所示：

```
buf.copy(targetBuffer[, targetStart[, sourceStart[, sourceEnd]]])
```

###### 参数
参数描述如下：
- targetBuffer - 要拷贝的 Buffer 对象。
- targetStart - 数字, 可选, 默认: 0
- sourceStart - 数字, 可选, 默认: 0
- sourceEnd - 数字, 可选, 默认: buffer.length
###### 返回值
没有返回值。
###### 实例

```
var buf1 = Buffer.from('abcdefghijkl');
var buf2 = Buffer.from('RUNOOB');

//将 buf2 插入到 buf1 指定位置上
buf2.copy(buf1, 2);

console.log(buf1.toString());
```

执行以上代码，输出结果为：

```
abRUNOOBijkl
```

##### 缓冲区裁剪
Node 缓冲区裁剪语法如下所示：

```
buf.slice([start[, end]])
```

###### 参数
参数描述如下：
- start - 数字, 可选, 默认: 0
- end - 数字, 可选, 默认: buffer.length
###### 返回值
返回一个新的缓冲区，它和旧缓冲区指向同一块内存，但是从索引 start 到 end 的位置剪切。
###### 实例

```
var buffer1 = Buffer.from('runoob');
// 剪切缓冲区
var buffer2 = buffer1.slice(0,2);
console.log("buffer2 content: " + buffer2.toString());
```

执行以上代码，输出结果为：

```
buffer2 content: ru
```

##### 缓冲区长度
###### 语法
Node 缓冲区长度计算语法如下所示：

```
buf.length;
```

###### 返回值
返回 Buffer 对象所占据的内存长度。
###### 实例

```
var buffer = Buffer.from('www.runoob.com');
//  缓冲区长度
console.log("buffer length: " + buffer.length);
```

执行以上代码，输出结果为：

```
buffer length: 14
```
#### Stream和Buffer
##### Stream
在一个应用程序中，流是一组有序的、有起点和终点的字节数据的传输手段。
##### Buffer
用于创建一个专门存放二进制数据的缓存区
###### 1.1 Stream

Stream 有四种流类型，且所有的 Stream 对象都是 EventEmitter 的实例： 
- Readable – 可读操作。 
- Writable – 可写操作。 
- Duplex – 可读可写操作. 
- Transform – 操作被写入数据，然后读出结果。

这意味着Stream对象有四种，同时每个Stream对象都有他们对应的事情。这些事件会在下文阐述。

###### 1.2 Buffer

NodeJS的Buffer详解：http://www.cnblogs.com/dwj0931-node/p/5397986.html

- Buffer的用途：文件传输或者大数据传输
- Buffer的字节管理
- Buffer对象与其他对象的转换
- Buffer的保存
###### Why

1. 在后端中应用，因为在后端与前端、后端的IO中，很大机会会出现文件太大，不能一次性读取的问题。因此在前端中使用的方法：整体读取后再进行操作，会导致程序的等待时间过长，因此，流操作（stream）便营运而生。

2. 在readable和writable的Stream之间筑起沟通，如果仅仅使用事件方法来进行的话，代码会显得很冗杂，因此需要出现pipe（管道）方法来进行。


```
readable.pipe(writable);
//返回值为writable的对象
```

1. 在数据传输过程中，如果需要把其中一段Stream截取并且进行修改查看，则需要Buffer类来进行协助，并且转化成其他的人类可理解对象
##### How

所有的 Stream 对象都是 EventEmitter 的实例。常用的事件有： 
- data – 当有数据可读时触发。 
- end – 没有更多的数据可读时触发。 
- error – 在接收和写入过程中发生错误时触发。 
- finish – 所有数据已被写入到底层系统时触发。 
- drain—缓冲区已满

1. 在文件流读取、写入中，有特定的事件可以提供监听。

2. 在读取到的二进制Bytes中，如何把字节重新解读成我们需要的数据 
通常传输的文件是图片、音频、视频等文件，中间程序要处理的有（百分比评估、是否传输完成、再一次传输给别的地方等），而不会在传输过程中要求打开修改文件（又不是迅雷那种边下载边看 

setEncoding和Buffer

3. 前端如何以流的方式发送信息给后端（socket）

    1. 可操作的流：socket、webSocket。
    
        以socket形式传输的流，前端是可以控制传输的量，并且得到反馈的。例如传输一张图片，用流（二进制）的方式来传输，可以精确到传输的百分比、断点续传等功能。

    2. 一次性的流：http 
    
        HTTP传输的Request和Response，则是一次性后端读取到一系列信息，但是后端在处理的时候，是完全可以用Readable Stream的形式来读取的。

##### Different

Stream与readFile、readFileSync 
Stream是每次读取一部分进入缓冲区，并且根据开发者定义的事件进行处理。 

而ReadFile、ReaderFileSync这些则是异步（或者同步）一次性把文件读取进入缓冲区，然后再进行操作。

##### Scene(应用场景）

###### 5.1 Stream的使用场景

大文件传输（不能短期阻塞完成的） 
文件需要多次传输的，使用pipe（防止代码过于冗杂）

###### 5.2 Buffer的使用场景

需要操作流的时候（大文件、大量数据）

##### nodejs buffer stream区别

###### buffer 
为数据缓冲对象，是一个类似数组结构的对象，可以通过指定开始写入的位置及写入的数据长度，往其中写入二进制数据。 
###### stream 
是对buffer对象的高级封装，其操作的底层还是buffer对象，stream可以设置为可读、可写，或者即可读也可写，在nodejs中继承了EventEmitter接口，可以监听读入、写入的过程。具体实现有文件流，httpresponse等

#### Node.js Buffer(缓冲区)和Stream流的关系
 JavaScript 语言自身只有字符串数据类型，没有二进制数据类型。处理文件流和处理TCP流（如文件之间传数据），必须使用到二进制数据，因此有了Buffer类，该类用来创建一个专门存放二进制数据的缓存区。

    以下为管道流实现文件传输的例子：


```
var fs = require("fs");  
  
// 创建一个可读流  
var readerStream = fs.createReadStream('input.txt');  
  
// 创建一个可写流  
var writerStream = fs.createWriteStream('output.txt');  
  
// 管道读写操作  
// 读取 input.txt 文件内容，并将内容写入到 output.txt 文件中  
readerStream.pipe(writerStream);  
  
console.log("程序执行完毕");
```

  pipe()方法实现文件数据的复制，个人认为pipe()封装了一些Buffer类的方法，如将可读流中的数据转化为二进制并存入缓冲区：Buffer.from("可读流中的数据")。整个程序好像以下过程：

```
const buf1=Buffer.from("input.txt中的数据")//将input.txt中的数据转华为二进制数据  
const buf2=Buffer.from("output.txt中的数据")//将output.txt中的数据转华为二进制数据  
buf2.copy(buf1);//复制buf1中的二进制数据  
buf1.toString()  
buf2.toString()//输出复制后output.txt的数据
```
#### vue组件之间的通信

**组件关系有下面三种：父-->子、子-->父、非父子**
##### 父组件向子组件传递数据
父组件一共需要做4件事

1.import son from './son.js' 引入子组件 son

2.在components : {"son"} 里注册所有子组件名称

3.在父组件的template应用子组件, <son></son>

4.如果需要传递数据给子组件,就在template模板里写 <son :num="number"></son>



```
// 1.引入子组件 
    import counter from './counter'     import son from './son'

// 2.在ccmponents里注册子组件    
components : {
        counter,
        son
    },

// 3.在template里使用子组件  
<son></son>

// 4.如果需要传递数据,也是在templete里写
 
  <counter :num="number"></counter>
```

子组件只需要做1件事

1.用props接受数据,就可以直接使用数据

2.子组件接受到的数据,不能去修改。如果你的确需要修改,可以用计算属性,或者把数据赋值给子组件data里的一个变量

```
// 1.用Props接受数据     
    props: [              
        'num'
        ],

// 2.如果需要修改得到的数据,可以这样写
   props: [           
            'num'
        ],  
        data () {       
            return {
                number : this.num
            }
        },
```
##### 子组件向父组件传递数据
子组件一共需要1件事情

在数据变化后,用$emit触发即可

// 1. 子组件在数据变化后,用$emit触发即可,第二个参数可以传递参数

```
methods: {
        increment(){                  
        this.number++                    
        this.$emit('changeNumber', this.number)},
        }
```
父组件一共需要做2件事情

在template里定义事件

在methods里写函数,监听子组件的事件触发


```
// 1. 在templete里应用子组件时,定义事件changeNumber
      <counter :num="number"                 
      @changeNumber="changeNumber"
      >
      </counter>

// 2. 用changeNumber监听事件是否触发
        methods: {
            changeNumber(e){            
                console.log('子组件emit了',e);      
                this.number = e
            },
        }
```
##### 第二种通信方式: eventBus

eventBus这种通信方式,针对的是非父子组件之间的通信,它的原理还是通过事件的触发和监听。

但是因为是非父子组件的关系,他们需要有一个中间组件来连接。

我是使用的通过在根组件,也就是#app组件上定义了一个所有组件都可以访问到的组件,具体使用方式如下

使用eventBus传递数据,我们一共需要做3件事情

1.给app组件添加Bus属性 (这样所有组件都可以通过this.$root.Bus访问到它,而且不需要引入任何文件)

2.在组件1里,this.$root.Bus.$emit触发事件

3.在组件2里,this.$root.Bus.$on监听事件

```
// 1.在main.js里给app组件,添加bus属性
import Vue from 'vue'new Vue({
  el: '#app',
  components: { App },  
  template: '<App/>',
  data(){   
        return {
            Bus : new Vue()
    }
  }
})
// 或者
**main.js**
let bus = new Vue()
Vue.prototype.bus = bus
```

```
// 2.在组件1里,触发,
//使用bus.$emit('事件名称'，'传入参数')，作为发送消息的那一方；
emitincrement(){
        this.number++
        this.$root.Bus.$emit('eventName', this.number)
    },
```

```
// 3.在组件2里,监听事件,接受数据
// 使用bus.$on('事件名称'，'回调函数')，作为接收消息的那一方
mounted(){    
    this.$root.Bus.$on('eventName', value => {        
        this.number = value
        console.log('busEvent');
    })
}
```
##### 第三种通信方式: 利用localStorage或者sessionStorage
这种通信比较简单,缺点是数据和状态比较混乱,不太容易维护。

- 通过window.localStorage.getItem(key) 获取数据
- 通过window.localStorage.setItem(key,value) 存储数据

*注意用JSON.parse() / JSON.stringify()* 做数据格式转换。
##### 第四种通信方式: 利用Vuex()

官方文档：http://vuex.vuejs.org
--vuex：主要用来集中式管理组件状态，（如组件显示/隐藏，增加/减少）

1)启动一个项目

2)安装vuex：cnpm install vuex -D

3）vuex提供了两个非常好的方法：
- mapActions():可以将所有methods里面的方法，进行打包。即对所有的事件(或我们的行为)进行管理
- mapGetters：获取所有的数据，对所有的数据进行管理

4）vuex的工作过程：
1. 当用户点击时，会调用increment函数(即用户有一个动作dispatch)
- 　　mapActions将函数(动作dispatch)提交到actions里面，并且传了commit这个参数(也是一个函数)
2. commit主要处理你要做什么，比如异步请求，判断，流程控制等，commit会将这些请求、状态提交到mutations里面
3. mutations主要用来处理状态(数据)的变化
4. mapGetters获取目前数据，将状态(数据)提交到getters上面，给mutations使用，让数据发生变化，
　　并返回(render)，从而更新视图

5）actions里面除了含有commit这的对象参数以外，还有另一个参数state(Vue组件中展示的数据源)

　　在这个过程中可以对数据进行判断，并作出相应的操作
　　

例子在src1/store.js中，这里是没有改写之前的代码

官方的文档中指出，vuex工作的各个过程是拆分开来实现的，下面我们就来进行一些分文件实现
项目的目录：

```
|--src文件夹
　　|--App.vue
　　|--main.js
　　|--store文件夹
　　　　|--index.js	//必须有index.js，它是我们组装模块并导出 store 的地方
　　　　|--actions.js	//是我们有动作触发之后，dispatch提交的地方
　　　　|--mutations.js	//commit提交的地方
　　　　|--types.js	//存放的是控制数据状态的地方，即控制数据如何变化
　　　　|--getters.js	//获取数据的目前状态，给mutations使用
```
#### axios中文文档

##### axios

基于http客户端的promise，面向浏览器和nodejs

##### 特色

浏览器端发起XMLHttpRequests请求

node端发起http请求

支持Promise API

监听请求和返回

转化请求和返回

取消请求

自动转化json数据

客户端支持抵御

##### 示例

###### 使用一个 GET 请求


```
//发起一个user请求，参数为给定的ID
axios.get('/user?ID=1234')
.then(function(respone){
    console.log(response);
})
.catch(function(error){
    console.log(error);
});

//上面的请求也可选择下面的方式来写
axios.get('/user',{
    params:{
        ID:12345
    }
})
    .then(function(response){
        console.log(response);
    })
    .catch(function(error){
        console.log(error)
    });
```

###### 发起一个 POST 请求


```
axios.post('/user',{
    firstName:'friend',
    lastName:'Flintstone'
})
.then(function(response){
    console.log(response);
})
.catch(function(error){
    console.log(error);
});
```

###### 发起一个多重并发请求


```
function getUserAccount(){
    return axios.get('/user/12345');
}

function getUserPermissions(){
    return axios.get('/user/12345/permissions');
}

axios.all([getUerAccount(),getUserPermissions()])
    .then(axios.spread(function(acc,pers){
        //两个请求现在都完成
    }));
```

##### axios API

axios 能够在进行请求时进行一些设置。

axios(config)


```
//发起 POST请求

axios({
    method:'post',
    url:'/user/12345',
    data:{
        firstName:'Fred',
        lastName:'Flintstone'
    }
});
axios(url[,config])
```

```
//发起一个GET请求
axios('/user/12345/);
```
##### 请求方法的重命名。

为了方便，axios提供了所有请求方法的重命名支持


```
axios.request(config)

axios.get(url[,config])

axios.delete(url[,config])

axios.head(url[,config])

axios.post(url[,data[,config]])

axios.put(url[,data[,config]])

axios.patch(url[,data[,config]])
```


**注意**

当时用重命名方法时 url , method ,以及 data 特性不需要在config中设置。

##### 并发 Concurrency

有用的方法

axios.all(iterable)

axios.spread(callback)

###### 创建一个实例

你可以使用自定义设置创建一个新的实例


```
axios.create([config])

var instance = axios.create({
    baseURL:'http://some-domain.com/api/',
    timeout:1000,
    headers:{'X-Custom-Header':'foobar'}
});
```

###### 实例方法

下面列出了一些实例方法。具体的设置将在实例设置中被合并。


```
axios#request(config)

axios#get(url[,config])

axios#delete(url[,config])

axios#head(url[,config])

axios#post(url[,data[,config]])

axios#put(url[,data[,config]])

axios#patch(url[,data[,config]])
```

##### 请求设置

以下列出了一些请求时的设置。只有 url 是必须的，如果没有指明的话，默认的请求方法是GET .

##### 返回响应概要 Response Schema

一个请求的返回包含以下信息


```
{
    //`data`是服务器的提供的回复（相对于请求）
    data{},

    //`status`是服务器返回的http状态码
    status:200,


    //`statusText`是服务器返回的http状态信息
    statusText: 'ok',

    //`headers`是服务器返回中携带的headers
    headers:{},

    //`config`是对axios进行的设置，目的是为了请求（request）
    config:{}
}
```

使用 then ，你会接受打下面的信息


```
axios.get('/user/12345')
    .then(function(response){
        console.log(response.data);
        console.log(response.status);
        console.log(response.statusText);
        console.log(response.headers);
        console.log(response.config);
    });
```

使用 catch 时，或者传入一个 reject callback 作为 then 的第二个参数，那么返回的错误信息将能够被使用。

##### 默认设置（Config Default)

你可以设置一个默认的设置，这设置将在所有的请求中有效。

##### 全局默认设置 Global axios defaults


```
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type']='application/x-www-form-urlencoded';
```

##### 实例中自定义默认值 Custom instance default


```
//当创建一个实例时进行默认设置
var instance = axios.create({
    baseURL:'https://api.example.com'
});

//在实例创建之后改变默认值
instance.defaults.headers.common['Authorization'] = AUTH_TOKEN;
```

##### 设置优先级 Config order of precedence

设置(config)将按照优先顺序整合起来。首先的是在 lib/defaults.js 中定义的默认设置，其次是 defaults 实例属性的设置，最后是请求中 config 参数的设置。越往后面的等级越高，会覆盖前面的设置。

看下面这个例子：


```
//使用默认库的设置创建一个实例，
//这个实例中，使用的是默认库的timeout设置，默认值是0。
var instance = axios.create();

//覆盖默认库中timeout的默认值
//此时，所有的请求的timeout时间是2.5秒
instance.defaults.timeout = 2500;

//覆盖该次请求中timeout的值，这个值设置的时间更长一些
instance.get('/longRequest',{
    timeout:5000
});
```

##### 拦截器 interceptors

你可以在 请求 或者 返回 被 then 或者 catch 处理之前对他们进行拦截。


```
//添加一个请求拦截器
axios.interceptors.request.use(function(config){
    //在请求发送之前做一些事
    return config;
},function(error){
    //当出现请求错误是做一些事
    return Promise.reject(error);
});

//添加一个返回拦截器
axios.interceptors.response.use(function(response){
    //对返回的数据进行一些处理
    return response;
},function(error){
    //对返回的错误进行一些处理
    return Promise.reject(error);
});
```

如果你需要在稍后移除拦截器,你可以


```
var myInterceptor = axios.interceptors.request.use(function(){/*...*/});
axios.interceptors.rquest.eject(myInterceptor);
```

你可以在一个axios实例中使用拦截器


```
var instance = axios.create();
instance.interceptors.request.use(function(){/*...*/});
```

##### 错误处理 Handling Errors


```
axios.get('user/12345')
    .catch(function(error){
        if(error.response){
            //存在请求，但是服务器的返回一个状态码
            //他们都在2xx之外
            console.log(error.response.data);
            console.log(error.response.status);
            console.log(error.response.headers);
        }else{
            //一些错误是在设置请求时触发的
            console.log('Error',error.message);
        }
        console.log(error.config);
    });
```

你可以使用 validateStatus 设置选项自定义HTTP状态码的错误范围。


```
axios.get('user/12345',{
    validateStatus:function(status){
        return status < 500;//当返回码小于等于500时视为错误
    }
});
```
##### 取消 Cancellation

你可以使用 cancel token 取消一个请求

axios的cancel token API是基于**cnacelable promises proposal**，其目前处于第一阶段。
你可以使用 CancelToke.source 工厂函数创建一个cancel token，如下：

```
var CancelToke = axios.CancelToken;
var source = CancelToken.source();

axios.get('/user/12345', {
    cancelToken:source.toke
}).catch(function(thrown){
    if(axiso.isCancel(thrown)){
        console.log('Rquest canceled', thrown.message);
    }else{
        //handle error
    }
});

//取消请求(信息参数设可设置的)
source.cancel("操作被用户取消");
```

你可以给 CancelToken 构造函数传递一个executor function来创建一个cancel token:


```
var CancelToken = axios.CancelToken;
var cancel;

axios.get('/user/12345', {
    cancelToken: new CancelToken(function executor(c){
        //这个executor 函数接受一个cancel function作为参数
        cancel = c;
    })
});

//取消请求
cancel();
```

注意：你可以使用同一个cancel token取消多个请求。