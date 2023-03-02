
# 真-前言
原文：[https://www.jianshu.com/p/9d04d5ea783d](https://www.jianshu.com/p/9d04d5ea783d) ，写于 18年 4 月初学 Node 的时候，到现在也快3年了，现在整理之前的文章，颇有些自己年级大了，还是年轻的时候学习精力旺盛的感觉。。。

---


#### 前言：
因为以前学习Node.js并没有真正意义上的去学习它，而是粗略的学习了npm的常用命令和Node.js一些模块化的语法，因此昨天花了一天的时间看了《Node.js开发指南》一书。通过这本书倒是让我对Node.js的认识更为全面，但由于这本书出版时间过早，有些API已经发生了变化或已经被废弃，而对于学习Node.js来说，核心部分又是最为重要的一环，因此我配合官方文档对这本书的第四章-Node.js核心进行了总结与梳理，由于水平有限，如有疏漏与错误，请指正。

## 全局对象

全局对象我想学过JavaScript的都知道在浏览器是window，在程序的任何地方都可以访问到全局对象，而在Node.js中，这个全局对象换成了global，所有的全局变量（除了global本身）都是global对象的属性。而我们在Node.js中能够直接访问的对象通常都是global的属性，如：console，process等。


### 全局对象与全局变量

global最根本的作用就是作为全局变量的宿主。按照ECMAScript规范，满足以下条件的变量即为全局变量：

- 在最外层定义的变量（在Node.js中不存在，因为Node.js的代码在模块中执行，不存在在最外层定义变量）
- 全局对象的属性
- 隐式定义的变量（即未定义而直接进行赋值的变量）

当我们定义一个全局变量的时候，这个全局变量会自动成为全局变量的属性。


### process

process 对象是一个全局变量，它提供当前 Node.js 进程的相关信息，以及控制当前 Node.js 进程。通常我们在写本地命令行程序的时候，少不了和它打交道。下面是它的最常用的成员方法：


#### 1.process.argv

process.argv 属性返回一个数组，这个数组包含了启动Node.js进程时的命令行参数。第一个元素为process.execPath，第二个元素为当前执行的JavaScript文件路径，剩余的元素为其他命令行参数。

例如存储一个名为argv.js的文件：

```
// print process.argv
process.argv.forEach((val, index) => {
  console.log(`${index}: ${val}`);
});
```

则命令行运行时这样的：

```
$ node process-args.js one two=three four

0: /usr/local/bin/node
1: /Users/mjr/work/node/process-args.js
2: one
3: two=three
4: four
```


#### 2.process.stdout

process.stdout是标准输出流，通常我们使用的console.log()输出打印字符，而process.stdout.write()函数提供了更为底层的接口。

```
process.stdout.write('请输入num1的值：');
```


#### 3.process.stdin

process.stdin是标准输入流，初始时它是暂停的，要想从标准输入读取数据，我们必须恢复流，并手动编写流的事件响应函数。

```
/*1:声明变量*/
var num1, num2;
/*2：向屏幕输出，提示信息，要求输入num1*/
process.stdout.write('请输入num1的值：');
/*3：监听用户的输入*/
process.stdin.on('data', function (chunk) {
    if (!num1) {
        num1 = Number(chunk);
        /*4：向屏幕输出，提示信息，要求输入num2*/
        process.stdout.write('请输入num2的值');
    } else {
        num2 = Number(chunk);
        process.stdout.write('结果是：' + (num1 + num2));
    }
});
```


#### 4.process.nextTick(callback[, ...args])

...args  调用 callback时传递给它的额外参数<br />process.nextTick()方法将 callback 添加到"next tick 队列"。 一旦当前事件轮询队列的任务全部完成，在next tick队列中的所有callbacks会被依次调用。<br />这种方式不是setTimeout(fn, 0)的别名。它更加有效率，因此别用setTimeout去代替process.nextTick。事件轮询随后的ticks 调用，会在任何I/O事件（包括定时器）之前运行。

```
console.log('start');
process.nextTick(() => {
  console.log('nextTick callback');
});
console.log('scheduled');

// start
// scheduled
// nextTick callback
```


### console

console 模块提供了一个简单的调试控制台，类似于 Web 浏览器提供的 JavaScript 控制台。该模块导出了两个特定的组件：

- 一个 Console 类，包含 console.log() 、 console.error() 和 console.warn() 等方法，可以被用于写入到任何 Node.js 流。
- 一个全局的 console 实例，可被用于写入到 process.stdout 和 process.stderr。 全局的 console 使用时无需调用 require('console')。(注意：全局的 console 对象的方法既不总是同步的（如浏览器中类似的 API），也不总是异步的（如其他 Node.js 流)。

比如全局下的常见的console实例：

```
console.log('hello，world');
// hello，world
console.log('hello%s', 'world');
// helloworld
console.error(new Error('错误信息'));
//  Error: 错误信息
const name = '描述';
console.warn(`警告${name}`);
// 警告描述
console.trace() // 向标准错误流输出当前的调用栈
```

常见的Console类：

```
const out = getStreamSomehow();
const err = getStreamSomehow();
const myConsole = new console.Console(out, err);

myConsole.log('hello，world');
// hello，world
myConsole.log('hello%s', 'world');
// helloworld
myConsole.error(new Error('错误信息'));
// Error: 错误信息
const name = '描述';
myConsole.warn(`警告${name}`);
//警告描述
```


## 常用工具  util

util 模块主要用于支持 Node.js 内部 API 的需求。 大部分实用工具也可用于应用程序与模块开发者，用于弥补核心JavaScript的功能的不足。它的可以这样调用：

```
const util = require('util');
```


### 1.util.inspect(object[, options])

util.inspect() 方法返回 object 的字符串表示，主要用于调试和错误输出。 附加的 options 可用于改变格式化字符串的某些方面。它至少接受一个参数objet，即要转换的参数，而option则是可选的，可选内容如下：

- showHidden  如果为 true，则 object 的不可枚举的符号与属性也会被包括在格式化后的结果中。 默认为 false。
- depth  指定格式化 object 时递归的次数。 这对查看大型复杂对象很有用。 默认为 2。 若要无限地递归则传入 null。
- colors  如果为 true，则输出样式使用 ANSI 颜色代码。 默认为 false，可自定义。
- customInspect  如果为 false，则 object 上自定义的 inspect(depth, opts) 函数不会被调用。 默认为 true。
- showProxy  如果为 true，则 Proxy 对象的对象和函数会展示它们的 target 和 handler 对象。 默认为 false。
- maxArrayLength  指定格式化时数组和 TypedArray 元素能包含的最大数量。 默认为 100。 设为 null 则显式全部数组元素。 设为 0 或负数则不显式数组元素。
- breakLength  一个对象的键被拆分成多行的长度。 设为 Infinity 则格式化一个对象为单行。 默认为 60。

例如，查看 util 对象的所有属性：

```
const util = require('util');
console.log(util.inspect(util, { showHidden: true, depth: null }));
```


### 2.util.callbackify(original)

util.callbackify(original)方法将 async 异步函数(或者一个返回值为 Promise 的函数)转换成遵循 Node.js 回调风格的函数。 在回调函数中, 第一个参数 err 为 Promise rejected 的原因 (如果 Promise 状态为 resolved , err为 null ),第二个参数则是 Promise 状态为 resolved 时的返回值。例如：

```
const util = require('util');

async function fn() {
  return await Promise.resolve('hello world');
}
const callbackFunction = util.callbackify(fn);

callbackFunction((err, ret) => {
  if (err) throw err;
  console.log(ret);
});
// hello world
```

注意：

- 回调函数是异步执行的, 并且有异常堆栈错误追踪. 如果回调函数抛出一个异常, 进程会触发一个 'uncaughtException' 异常, 如果没有被捕获, 进程将会退出。
- null 在回调函数中作为一个参数有其特殊的意义, 如果回调函数的首个参数为 Promise rejected 的原因且带有返回值, 且值可以转换成布尔值 false, 这个值会被封装在 Error 对象里, 可以通过属性 reason 获取。

```
function fn() {
  return Promise.reject(null);
}
const callbackFunction = util.callbackify(fn);

callbackFunction((err, ret) => {
    // 当Promise的rejecte是null时，它的Error与原始值都会被存储在'reason'中。
  err && err.hasOwnProperty('reason') && err.reason === null;  // true
});
```


## 事件驱动  events

events是Node.js最重要的模块，原因是Node.js本身架构就是事件式的，大多数 Node.js 核心 API 都采用惯用的异步事件驱动架构，而它提供了唯一的接口，因此堪称Node.js事件编程的及时。events 模块不仅用于用户代码与 Node.js 下层事件循环的交互，还几乎被所有的模块依赖。

所有能触发事件的对象都是 EventEmitter 类的实例。 这些对象开放了一个 eventEmitter.on() 函数，允许将一个或多个函数绑定到会被对象触发的命名事件上。 事件名称通常是驼峰式的字符串，但也可以使用任何有效的 JavaScript 属性名。对于每个事件， EventEmitter支持<br />若干个事件监听器。当事件发射时，注册到这个事件的事件监听器被依次调用，事件参数作<br />为回调函数参数传递。

例如：

```
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();
// eventEmitter.on() 方法用于注册监听器
myEmitter.on('event', () => {
  console.log('触发了一个事件！');
});
// eventEmitter.emit() 方法用于触发事件
myEmitter.emit('event');
```

下面让我们来看看EventEmitter最常用的API：

- EventEmitter.on(event, listener) 方法可以添加 listener 函数到名为 eventName 的事件的监听器数组的末尾。不会检查 listener 是否已被添加。多次调用并传入相同的 eventName 和 listener 会导致 listener 被添加与调用多次。
- emitter.prependListener(eventName, listener)方法可以添加 listener 函数到名为 eventName 的事件的监听器数组的开头。 不会检查 listener 是否已被添加。 多次调用并传入相同的 eventName 和 listener 会导致 listener 被添加与调用多次。
- eventEmitter.emit() 方法允许将任意参数传给监听器函数。当一个普通的监听器函数被 EventEmitter 调用时，标准的 this 关键词会被设置指向监听器所附加的 EventEmitter。

```
// 实例：
const myEE = new EventEmitter();
myEE.on('foo', () => console.log('a'));
myEE.prependListener('foo', () => console.log('b'));
myEE.emit('foo');
// 打印:
//   b
//   a
```

- EventEmitter.once(event, listener)  为指定事件注册一个单次监听器，即监听器最多只会触发一次，触发后立刻解除该监听器。

```
server.once('connection', (stream) => {
  console.log('首次调用！');
});
```

- EventEmitter.removeListener(event, listener) 移除指定事件的某个监听器， listener 必须是该事件已经注册过的监听器。（注意：removeListener 最多只会从监听器数组里移除一个监听器实例。 如果任何单一的监听器被多次添加到指定 eventName 的监听器数组中，则必须多次调用 removeListener 才能移除每个实例。）

```
const callback = (stream) => {
  console.log('有连接！');
};
server.on('connection', callback);
// ...
server.removeListener('connection', callback);
```

- EventEmitter.removeAllListeners([event]) 移除所有事件的所有监听器，如果指定  event ，则移除指定事件的所有监听器

```
const callback = (stream) => {
  console.log('有连接！');
};
server.on('connection', callback);
// ...
server.removeListener('connection', callback);
```


### error 事件

EventEmitter 定义了一个特殊的事件  error ，它包含了“错误”的语义，我们在遇到异常的时候通常会发射 error 事件。当 error被发射时，EventEmitter规定如果没有响<br />应的监听器，Node.js 会把它当作异常，退出程序并打印调用栈。我们一般要为会发射 error 事件的对象设置监听器，避免遇到错误后整个程序崩溃。

```
var events = require('events');
var emitter = new events.EventEmitter();
emitter.emit('error');
```


### 继承EventEmitter

大多数情况下，我们不会直接使用EventEmitter，而是在对象中继承它，包括fs,http在内的只要是支持事件响应的核心模块都是EventEmitter的子类。这样做的原因有以下两个：

- 具有某个实体功能的对象实现事件符合语义，事件的监听和发射应该是一个对象的方法。
- JavaScript 的对象机制是基于原型的，支持部分多重继承，继承  EventEmitter 不会打乱对象原有的继承关系。

目录：<br />[Node.js核心入门(一)](https://juejin.im/entry/5ac0e2265188255c272212d4)

- 全局对象
- 常用工具
- 事件机制

Node.js核心入门（二）

- 文件系统访问
- HTTP服务器与客户端


## 文件系统 fs

fs 模块是文件操作的封装，它提供了文件的读取、写入、更名、删除、遍历目录、链接等 POSIX 文件系统操作，且所有的方法都有异步和同步的形式。异步方法的最后一个参数都是一个回调函数。 传给回调函数的参数取决于具体方法，但回调函数的第一个参数都会保留给异常。 如果操作成功完成，则第一个参数会是 null 或 undefined。

```
const fs = require('fs');
fs.unlink('/tmp/hello', (err) => {
  if (err) throw err;
  console.log('成功删除 /tmp/hello');
});
```

当使用同步方法时，任何异常都会被立即抛出。 可以使用 try/catch 来处理异常，或让异常向上冒泡。

```
const fs = require('fs');
fs.unlinkSync('/tmp/hello');
console.log('成功删除 /tmp/hello');
```


### 1.fs.readFile(path,[options], callback)

fs.readFile(path,[options], callback) 是最简单的读取。它接受一个必选参数filename，表示要读取的文件名。第二个参数options是可选的，表示文件的字符编码。 callback 是回调函数，用于接收文件的内容。如果不指定 options ，则  callback 就是第二个参数。回调函数提供两个参数  err 和  data ， err 表示有没有错误发生，data 是文件内容。如果指定了  options， data 是一个解析后的字符串，否则  data 将会是以  Buffer形式表示的二进制数据。例如：

```
fs.readFile('/etc/passwd', 'utf8', callback);
```

需要注意的是，当 path 是一个目录时，fs.readFile() 与 fs.readFileSync() 的行为与平台有关。在 macOS、Linux 与 Windows 上，会返回一个错误。在 FreeBSD 上，会返回目录内容的表示。

```
// 在 macOS、Linux 与 Windows 上：
fs.readFile('<directory>', (err, data) => {
  // => [Error: EISDIR: illegal operation on a directory, read <directory>]
});

//  在 FreeBSD 上：
fs.readFile('<directory>', (err, data) => {
  // => null, <data>
});
```


### 2.fs.readFileSync(path[, options])

fs.readFileSync(filename, [encoding]) 是  fs.readFile 同步的版本。它接受的参数和  fs.readFile 相同，但读取到的文件内容会以函数返回值的形式返回。如果有错<br />误发生， fs 将会抛出异常，这时候我们就需要使用  try 和  catch 捕捉并处理异常。


### 3.fs.open(path, flags[, mode], callback)

fs.open(path, flags[, mode], callback)是 POSIX  open 函数的<br />封装，与 C 语言标准库中的  fopen 函数类似。它接受两个必选参数， path 为文件的路径，<br />而flags 可以是以下值：

```
'r' - 以读取模式打开文件。如果文件不存在则发生异常。

'r+' - 以读写模式打开文件。如果文件不存在则发生异常。

'rs+' - 以同步读写模式打开文件。命令操作系统绕过本地文件系统缓存。

（这对 NFS 挂载模式下打开文件很有用，因为它可以让你跳过潜在的旧本地缓存。 它对 I/O 的性能有明显的影响，所以除非需要，否则不要使用此标志。

注意，这不会使 fs.open() 进入同步阻塞调用。 如果那是你想要的，则应该使用 fs.openSync()。）

'w' - 以写入模式打开文件。文件会被创建（如果文件不存在）或截断（如果文件存在）。

'wx' - 类似 'w'，但如果 path 存在，则失败。

'w+' - 以读写模式打开文件。文件会被创建（如果文件不存在）或截断（如果文件存在）。

'wx+' - 类似 'w+'，但如果 path 存在，则失败。

'a' - 以追加模式打开文件。如果文件不存在，则会被创建。

'ax' - 类似于 'a'，但如果 path 存在，则失败。

'a+' - 以读取和追加模式打开文件。如果文件不存在，则会被创建。

'ax+' - 类似于 'a+'，但如果 path 存在，则失败。
```

mode 可设置文件模式（权限和 sticky 位），但只有当文件被创建时才有效。默认为 0o666，可读写。


### 4.fs.read(fd, buffer, offset, length, position, callback)

fs.read(fd, buffer, offset, length, position, callback) 是 POSIX  read 函数的封装，相比  fs.readFile 提供了更底层的接口。从 fd 指定的文件中读取数据。buffer 是数据将被写入到的 buffer。offset 是 buffer 中开始写入的偏移量。length是一个整数，指定要读取的字节数。position指定从文件中开始读取的位置。如果position为null，则数据从当前文件读取位置开始读取，且文件读取位置会被更新。 如果 position 为一个整数，则文件读取位置保持不变。回调有三个参数 (err, bytesRead, buffer)。

```
var fs = require('fs');
fs.open('content.txt', 'r', function(err, fd) {
if (err) {
console.error(err);
return;
}
var buf = new Buffer(8);
fs.read(fd, buf, 0, 8, null, function(err, bytesRead, buffer) {
if (err) {
console.error(err);
return;
}
console.log('bytesRead: ' + bytesRead);
console.log(buffer);
})
});
```

输出：

```
bytesRead: 8
<Buffer 54 65 78 74 20 e6 96 87>
```


## HTTP服务器与客户端

Node.js 标准库提供了http模块，其中封装了一个高效的 HTTP 服务器和一个简易的HTTP 客户端。 http.Server 是一个基于事件的 HTTP 服务器，它的核心由 Node.js 下层 C++<br />部分实现，而接口由 JavaScript 封装，兼顾了高性能与简易性。 http.request则是一个HTTP 客户端工具，用于向 HTTP 服务器发起请求，例如实现 Pingback或者内容抓取。

Node.js 中的HTTP接口被设计成支持协议的许多特性。比如，大块编码的消息。这些接口不缓冲完整的请求或响应，用户能够以流的形式处理数据。HTTP消息头由一个对象表示，其中键名是小写的，键值不能修改：

```
{ 'content-length': '123',
  'content-type': 'text/plain',
  'connection': 'keep-alive',
  'host': 'mysite.com',
  'accept': '*/*' }
```

为了支持各种可能的 HTTP 应用，Node.js的 HTTP API是非常底层的。它只涉及流处理与消息解析。它把一个消息解析成消息头和消息主体，但不解析具体的消息头或消息主体。键名是小写的，键值不能修改。为了支持各种可能的 HTTP 应用，Node.js 的 HTTP API 是非常底层的。 它只涉及流处理与消息解析。 它把一个消息解析成消息头和消息主体，但不解析具体的消息头或消息主体。


### HTTP服务器

http.Server 是  http  模块中的 HTTP 服务器对象，用 Node.js 做的所有基于 HTTP 协议的系统，如网站、社交应用甚至代理服务器，都是基于http.Server实现的。它提供了一套封装级别很低的API，仅仅是流控制和简单的学习解析，而所有的高级功能都是通过它的接口来实现的。比如官网上的这个例子：

```
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World
');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
})
```

在这段代码中，就使用了http.createServer([requestListener])来新建一个的 http.Server 实例。现在就先让我们来看看http.createServer([requestListener])吧。


#### 1. http.Server 的事件

http.Server 是一个基于事件的 HTTP 服务器，所有的请求都被封装为独立的事件，开发者只需要对它的事件编写响应函数即可实现 HTTP 服务器的所有功能。它继承自<br />EventEmitter ，提供了以下几个事件：

- request：每次接收到一个请求时触发。 注意，每个连接可能有多个请求（在 HTTP keep-alive 连接的情况下）。
- connection ：当一个新的 TCP 流被建立时触发。socket 是一个 net.Socket 类型的对象。 通常用户无需访问该事件。 注意，因为协议解析器绑定到 socket 的方式，socket 不会触发 'readable' 事件。socket 也可以通过 request.connection 访问。
- connect：每当客户端发送 HTTP CONNECT 请求时触发。 如果该事件未被监听，则发送 CONNECT 请求的客户端会关闭连接。当该事件被触发后，请求的 socket 上没有 'data' 事件监听器，这意味着需要绑定 'data' 事件监听器，用来处理 socket 上被发送到服务器的数据。
- close：当服务器关闭时，该事件被触发。注意不是在用户连接断开时，而是服务器关闭时。

在这些事件最常用的是request是最常用的，因此  http 提供了一个捷径：<br />http.createServer([requestListener]) ，功能是创建一个 HTTP 服务器并将requestListener 作为  request 事件的监听函数。我们上面那个官网的例子就是如此，其实它显式的实现方法是这样的：

```
//httpserver.js
const http = require('http');
const hostname = '127.0.0.1';
const port = 3000;
const server = new http.Server();
server.on('request', (req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World
');
});
server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
})
```


#### 2. http.ServerRequest

http.ServerRequest  是 HTTP 请求的信息，是后端开发者最关注的内容。它一般由http.Server 的 request 事件发送，作为第一个参数传递，通常简称  request 或 HTTP 请求一般可以分为两部分：请求头（Request Header）和请求体（Requset Body）。以上内容由于长度较短都可以在请求头解析完成后立即读取。而请求体可能相对较长，需要一定的时间传输，因此  http.ServerRequest  提供了以下3个事件用于控制请求体传输。req。HTTP请求一般可以分为两部分：请求头（RequestHeader）和请求体（RequsetBody）。以上内容由于长度较短都可以在请求头解析完成后立即读取。而请求体可能相对较长，需要一定的时间传输，因此http.ServerRequest提供了以下3个事件用于控制请求体传输。http.ServerRequest提供了3个事件用于控制请求体传输：

1. data：当请求体数据到来时，该事件被触发，提供一个参数给回调函数，是接受到的数据，该事件可能被多次调用（所有data按顺序的集合，是请求体数据）。如果该事件没有被监听，请求体将被抛弃；
2. end：当请求体数据完成时该事件触发。此后不再触发data事件；
3. close：用户当前请求结束时，该事件被触发。不同于end，如果用户强制终止了传输，也还是调用close。
```
              表4-2  ServerRequest 的属性
     名 称                  含 义
     complete        客户端请求是否已经发送完成
     httpVersion     HTTP 协议版本，通常是 1.0 或 1.1
     method          HTTP 请求方法，如 GET、POST、PUT、DELETE 等
     url             原始的请求路径，例如 /static/image/x.jpg 或 /user?name=byvoid
     headers         HTTP 请求头
     trailers        HTTP 请求尾（不常见）
     connection      当前 HTTP 连接套接字，为 net.Socket 的实例
     socket          connection 属性的别名
     client          client 属性的别名
```


#### 3. 获取 GET 请求内容

注意， http.ServerRequest 提供的属性中没有类似于 PHP 语言中的 $_GET 或 $_POST 的属性，GET请求被直接内嵌在路径中。URL是完整的请求路径（包括？后面的部分），因此手动解析后面的内容作为GET请求的参数。Node.js的url模块中的parse函数提供了这个功能。

以url：[http://127.0.0.1/user?name=byvoid&email=byvoid@byvoid.com](http://127.0.0.1/user?name=byvoid&email=byvoid@byvoid.com)为例：

```
var http = require("http");
var url = require("url");
var server = new http.Server();
server.on("request", function (req, res) {
    if (req.url == "/favicon.ico") {
        return;
    }
    var m = url.parse(req.url, true);
    console.log(m)
    res.writeHead(200, {'Content-type': 'text/html;charset = utf8'});
    res.end();
})
server.listen(80);
console.log("The server begin");
```

console.log输出内容：

```
Url {
  protocol: null,
  slashes: null,
  auth: null,
  host: null,
  port: null,
  hostname: null,
  hash: null,
  search:'?name=byvoid&email=byvoid@byvoid.com',
  query: { name: 'byvoid', email:'byvoid@byvoid.com' },
  pathname: '/user',
  path:'/user?name=byvoid&email=byvoid@byvoid.com',
  href:'/user?name=byvoid&email=byvoid@byvoid.com' 
}
```


#### 4. 获取 POST 请求内容

HTTP 协议1.1版本提供了8种标准的请求方法，而其中最常见的就是 GET 和 POST。相比GET请求把所有的内容编码到访问路径中，POST 请求的内容全部都在请求体中。http.ServerRequest 并没有一个属性内容是在请求体中，原因是等待请求体传输可能是一件耗时的工作，譬如上传文件。而很多时候我们可能并不需要理会请求体的内容，且恶意的 POST<br />请求会大大消耗服务器的资源。所以 Node.js 默认是不会解析请求体的，因此当我们需要的时候，我们就要手写一个，具体实现方法如下：

```
var http = require('http');
var querystring = require('querystring');
var util = require('util');
http.createServer(function(req, res) {
    var post = '';
    req.on('data', function(chunk) {
    post += chunk;
});
req.on('end', function() {
    post = querystring.parse(post);
    res.end(util.inspect(post));
    });
}).listen(3000);
```


#### 5.http.ServerResponse

http.ServerResponse  是返回给客户端的信息，决定了用户最终能看到的结果。它也是由  http.Server 的  request  事件发送的，作为第二个参数传递，一般简称为<br />response 或 res 。http.ServerResponse 有三个重要的成员函数，用于返回响应头、响应内容以及结束请求：

- response.writeHead(statusCode, [headers]) ：向请求的客户端发送响应头。statusCode是HTTP状态码，如200（请求成功）、404（未找到）等。headers是一个类似关联数组的对象，表示响应头的每个属性。该函数在一个请求内最多只能调用一次，如果不调用，则会自动生成一个响应头。
- response.write(data, [encoding]) ：向请求的客户端发送响应内容。 data 是一个  Buffer 或字符串，表示要发送的内容。如果  data 是字符串，那么需要指定<br />encoding 来说明它的编码方式，默认是 utf-8 。在 response.end 调用之前，response.write  可以被多次调用。
- response.end([data], [encoding]) ：结束响应，告知客户端所有发送已经完成。当所有要返回的内容发送完毕的时候，该函数 必须 被调用一次。它接受两个可选参数，意义和 response.write  相同。如果不调用该函数，客户端将永远处于等待状态。


### HTTP 客户端

http 模块提供了两个函数 http.request和http.get，功能是作为客户端向HTTP服务器发起请求。


#### 1.http.request(options,callback)

http.request(options,callback)发起HTTP请求，它接受两个参数，option是一个类似关联数组的对象，表示请求的参数，callback是请求的回调函数。option常用的参数如下所示：

- host ：请求网站的域名或 IP 地址。
- port ：请求网站的端口，默认 80。
- method ：请求方法，默认是 GET。
- path ：请求的相对于根的路径，默认是“ / ”。 QueryString  应该包含在其中。例如  /search?query=byvoid 。
- headers ：一个关联数组对象，为请求头的内容。

而callback 则传递一个参数，为 http.ClientResponse 的实例。http.request 返回一个http.ClientRequest 的实例，下面是一个通过 http.request  发送 POST 请求的代码：

```
var http = require('http');
var querystring = require('querystring');
var contents = querystring.stringify({
    name: 'byvoid',
    email: 'byvoid@byvoid.com',
    address: 'Zijing 2#, Tsinghua University',
});
var options = {
    host: 'www.byvoid.com',
    path: '/application/node/post.php',
    method: 'POST',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Content-Length' : contents.length
    }
};
var req = http.request(options, function(res) {
    res.setEncoding('utf8');
    res.on('data', function (data) {
    console.log(data);
    });
});
req.write(contents);
req.end();
```

运行结果如下：

```
array(3) {
["name"]=>
string(6) "byvoid"
["email"]=>
string(17) "byvoid@byvoid.com"
["address"]=>
string(30) "Zijing 2#, Tsinghua University"
}
```


#### 2.http.get(options, callback)

http 模块还提供了一个更加简便的方法用于处理GET请求：http.get(options, callback)。它是http.request的简化版，唯一的区别在于http.get自动将请求方法设为了 GET 请求，同时不需要手动调用 req.end() ：

```
var http = require('http');
http.get({host: 'www.byvoid.com'}, function(res) {
    res.setEncoding('utf8');
    res.on('data', function (data) {
    console.log(data);
    });
});
```


#### http.ClientRequest

该对象在 http.request() 内部被创建并返回。它表示着一个正在处理的请求，其请求头已进入队列。它提供一个response事件，即http.request或http.get第二个参数指定的回调函数的绑定对象。

```
var http = require('http');
var req = http.get({host: 'www.byvoid.com'});
    req.on('response', function(res) {
    res.setEncoding('utf8');
    res.on('data', function (data) {
    console.log(data);
    });
});
```

http.ClientRequest像http.ServerResponse一样也提供了  write 和 end 函数，用于向服务器发送请求体，通常用于 POST、PUT 等操作。所有写结束以后必须调用 end<br />函数以通知服务器，否则请求无效。http.ClientRequest  还提供了以下常用的函数:

- request.abort() ：标记请求为终止。 调用该方法将使响应中剩余的数据被丢弃且 socket 被销毁。
- request.setTimeout(timeout,[callback])：设置请求超时时间， timeout为毫秒数。一旦socket被分配给请求且已连接，socket.setTimeout() 会被调用。
- request.end([data[, encoding]][, callback])结束发送请求。如果部分请求主体还未被发送，则会刷新它们到流中。 如果请求是分块的，则会发送终止字符 '0\r
\r
'。


#### http.ClientResponse

http.ClientResponse 与  http.ServerRequest 相似，提供了三个事件 data 、end和 close，分别在数据到达、传输结束和连接结束时触发，其中  data 事件传递一个参数chunk ，表示接收到的数据。

http.ClientResponse  也提供了一些属性，用于表示请求的结果状态：

```
statusCode   HTTP 状态码，如 200、404、500
httpVersion  HTTP 协议版本，通常是 1.0 或 1.1
headers      HTTP 请求头
trailers     HTTP 请求尾（不常见）
```

http.ClientResponse 还提供了以下几个特殊的函数：

- response.setEncoding([encoding]) ：设置默认的编码，当  data 事件被触发时，数据将会以encoding编码。默认值是null，即不编码，以  Buffer 的形式存储。常用编码为 utf8。
- response.pause()：暂停接收数据和发送事件，方便实现下载功能。
- response.resume() ：从暂停的状态中恢复。
