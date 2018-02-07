---
layout: post
title: "nodeJs文件读取"
date: 2018-02-08
categories: [nodeJs文件读取]
datetime: 2018-02-08
tags: [nodeJs文件读取]

---
* content
{:toc}
nodejs中文件读取的基本操作以及部分对Stream的理解
<!-- more -->


> nodeJs的文件读写是依赖fs模块来实现的。

## 使用

通过 require('fs') 使用该模块。 所有的方法都有异步和同步的形式。
异步方法的最后一个参数都是一个回调函数。 传给回调函数的参数取决于具体方法，但回调函数的第一个参数都会保留给异常。 如果操作成功完成，则第一个参数会是 null 或 undefined。


- fs.readFile(filename, callback)
异步读取文件。

filename是读取文件的文件名，如果是相对路径，则通过当前进程执行的路径来查找文件。

回调函数有两个参数callback(err, buffer)

第一个参数为err(如果没有报错，该参数值为null)，进行操作时，应先判断err是否有值。

第二个参数是代表文件内容的Buffer实例。

- fs.writeFile(filename, content [, encode], callback)

异步写入文件，content为写入内容字符串, encode为编码(默认utf8)

对应的cb回调函数中只有err一个参数(若无报错为null);

> 这两个方法有相应的同步版本:

- fs.readFileSync(filename [,encode])

第二个参数可以是表示编码的字符串，也可以是一个配置对象({encoding: null, flag: 'r'})

即默认编码是null，读取模式为r(只读)。

如果不指定编码方式，fs返回一个表示文件内容的Buffer实例，否则返回字符串。

- fs.writeFileSync(filename, content, encode)

filename表示文件名，content是要写入的字符串内容，encode是文件内容编码方式。

通过这几个接口我们可以完成简单的文件读写工作：
```js
const fs = require('fs');
const txt = fs.readFileSync('/txt.txt', {encoding: 'utf8'});
 
fs.writeFileSync('/newtxt.txt', source); 

```
上面几个接口，无论是异步或是同步，都是等文件读取完成，才进行操作的。

如果操作对象是一些小文件，这种操作没有什么问题。

但是在服务器端，文件体积一般很庞大，用这种方式效率低下，导致线程阻塞，

甚至会因为内存不足而崩溃。

这里要引用Stream流的概念。

## Stream

nodejs中Stream是EventEmitter的实现，你可以理解为在程序后台打开了一个文件(不占用主线程)，

程序会一点一点的读取(写入)文件，通过事件和回调来完成文件的读写。来看个例子

```js
const fs = require('fs');
// 创建一个可读流
const readStream = fs.createReadStream('/txt.txt');
const writeStream = fs.createWriteStream('/newtxt.txt');
 
readStream.on('data', function(chunk){
  // 可读流收到data事件时，将内容写入到可写流
  writeStream.write(chunk);
});
 
readStream.on('end', function(){
  // 当可读流读取完成，会发出end事件，这时我们要把可写流关闭
  writeStream.end();
});

```
当调用fs.createReadStream时，相当于创建了一个文件读取流。

可以通过监听data事件，来获取读取的部分数据，来进行一些操作。

