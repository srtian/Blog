
![](https://cdn.nlark.com/yuque/0/2020/svg/296173/1586333536866-f558ad17-fd98-42ff-980a-8fe22828c820.svg)

## 一、模块
Node.js模块分为三种：

- require
- export
- module

### 1.1、require
require1函数用于在当前模块中加载和使用别的模块，传入一个模块名，返回一个模块导出对象。模块名可以使用相对路径（./）以及绝对路径（以`/`或`C:`之类的盘符开头）

```javascript
// foo1至foo4中保存的是同一个模块的导出对象
var foo1 = require('./foo');
var foo2 = require('./foo.js');
var foo3 = require('/home/user/foo');
var foo4 = require('/home/user/foo.js');
```

### 1.2、export
export 对象是当前模块的导出对象，用于导出模块的共有方法和属性。别的模块通过require函数使用当前模块时得到的就是当前模块的export对象：

```javascript
exports.sayhai = () => {
  console.log("hi, node");
};
console.log(1);
```

### 1.3、module
通过module对象可以访问当前模块的一些相关信息，但用途最多的当属替换当前模块的导出对象：
```javascript
module.exports = function () {
    console.log('Hello node!');
};
```
以上代码，便将默认导出对象被替换为一个函数

### 1.4、模块初始化
一个模块的JS代码仅在模块第一次被使用时执行一次，并在执行过程中初始化模块的导出对象，之后，缓存起来的导出对象被重复利用

```javascript
// example.js
let count = 0;

const add = () => {
  return ++count;
};

exports.add = add;

// app.js
var example1 = require('./example');
var example2 = require('./example');

console.log(example1.add());
console.log(example2.add());
console.log(example1.add());

// 输出：1 2 3
```
由输出我们可以得出example.js并没有被require初始化两次。
> 具体的关于模块的记录：[https://www.yuque.com/srtian/fe/agw824#TZgDs](https://www.yuque.com/srtian/fe/agw824#TZgDs)


## 二、代码的组织和部署

### 2.1、模块路径解析规则
上面已经知晓，require函数支持相对路径以及绝对路径，但这两种路径在模块之间建立了强耦合关系，一旦某个文件存放位置需要变更，使用该模块的其他模块的代码也需要调整。因此，require函数支持第三种路径，类似于 `foo/bar` 并依次按照下列规则解析路径，直到找到模块位置。

1. 内置模块：如果传递给require函数的是Node.js的内置模块名称，不做路径解析，直接返回内部模块的导出对象
2. node_modules目录：Node.js定义了一个特殊的 `node_modules` 目录存放模块。例如在该模块中使用`require('foo/bar')`方式加载模块时，则NodeJS依次尝试使用以下路径：

```javascript
 /home/user/node_modules/foo/bar
 /home/node_modules/foo/bar
 /node_modules/foo/bar
```

3. NODE_PATH环境变量：与PATH环境变量类似，NodeJS允许通过NODE_PATH环境变量来指定额外的模块搜索路径。NODE_PATH环境变量中包含一到多个目录路径，路径之间在Linux下使用`:`分隔，在Windows下使用`;`分隔。例如定义了以下NODE_PATH环境变量：
```
NODE_PATH=/home/user/lib:/home/lib
```
当使用`require('foo/bar')`的方式加载模块时，则NodeJS依次尝试以下路径。
```
/home/user/lib/foo/bar
/home/lib/foo/bar
```

### 2.2、包
JS模块的基本单位是单个的JS文件，但复杂的模块往往会由多个子模块组成。为了方便管理以及使用，我们可以将多个子模块组成的大模块称之为包，并把所有子模块放在同一个目录里。

在组成一个包的所有子模块中，需要有一个入口模块，入口模块的导出对象被称为包的导出对象：

```
- /home/user/lib/
    - cat/
        head.js
        body.js
        main.js
```
上面的cat目录就定义了一个包，包括三个自模块，main作为入口模块：

```javascript
var head = require('./head');
var body = require('./body');

exports.create = function (name) {
    return {
        name: name,
        head: head.create(),
        body: body.create()
    };
};
```
在其他模块中使用包的时候，就需要加载包的入口模块，使用`require('/home/user/lib/cat/main')`能达到目的，但是入口模块名称出现在路径里看上去不是个好主意。因此我们需要做点额外的工作，让包使用起来更像是单个模块。因此我们可以让模块的文件名为index.js，这样加载模块时可以使用模块所在目录的路径代替模块文件路径：

```javascript
// 等价的：
var cat = require('/home/user/lib/cat');
var cat = require('/home/user/lib/cat/index');
```
此外我们可以使用package.json文件来指定入口模块的路径，比如这样：

```
// 路径
- /home/user/lib/
    - cat/
        + doc/
        - lib/
            head.js
            body.js
            main.js
        + tests/
        package.json

// package.json
{
    "name": "cat",
    "main": "./lib/main.js"
}
```
如此一来，就同样可以使用`require('/home/user/lib/cat')`的方式加载模块。NodeJS会根据包目录下的`package.json`找到入口模块所在位置。


### 2.3 NPM
NPM能解决很多Node.js代码部署上的问题，使用场景如下：

- 允许用户从NPM服务器下载别人编写的三方包到本地使用。<br />
- 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。<br />
- 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。<br />

npm常用命令：

- NPM提供了很多命令，例如install和publish，使用npm help可查看所有命令。
- 使用npm help 可查看某条命令的详细帮助，例如npm help install。
- 在package.json所在目录下使用npm install . -g可先在本地安装当前命令行程序，可用于发布前的本地测试。
- 使用npm update 可以把当前目录下node_modules子目录里边的对应模块更新至最新版本。
- 使用npm update  -g可以把全局安装的对应命令行程序更新至最新版。
- 使用npm cache clear可以清空NPM本地缓存，用于对付使用相同版本号发布新版本代码的人。
- 使用npm unpublish @可以撤销发布自己发布过的某个版本代码。


## 三、文件操作

### 3.1、文件拷贝

#### 小文件拷贝
对于小文件的拷贝，我们可以使用fs.readFileSync从源路径读取内容，并使用fs.writeFileSync将文件内容写入目标路径。
> `process`是一个全局变量，可通过`process.argv`获得命令行参数。由于`argv[0]`固定等于NodeJS执行程序的绝对路径，`argv[1]`固定等于主模块的绝对路径，因此第一个命令行参数从`argv[2]`这个位置开始。

```javascript
var fs = require('fs');

function copy(src, dst) {
    fs.writeFileSync(dst, fs.readFileSync(src));
}

function main(argv) {
    copy(argv[0], argv[1]);
}

main(process.argv.slice(2));
```

#### 大文件拷贝
对于大文件，我们就不能一次将所有文件内容都读取到内存中然后一次写入磁盘，会导致内存爆仓。因此，我们只能读一点写一点，直到完成文件拷贝：

```javascript
var fs = require('fs');

function copy(src, dst) {
    fs.createReadStream(src).pipe(fs.createWriteStream(dst));
}

function main(argv) {
    copy(argv[0], argv[1]);
}

main(process.argv.slice(2));
```
我们使用fs.createReadStream创建一个源文件的读取数据流，并使用fs.createWriteStream创建一个目标文件的只写数据流，并且用pipi方法将两个数据流连接起来。

### 3.2、API

#### Buffer（数据块）
`Buffer`将JS的数据处理能力从字符串扩展到了任意二进制数据
```javascript
var bin = new Buffer([ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]);
bin[0]; // => 0x68;
var str = bin.toString('utf-8'); // => "hello"
var bin = new Buffer('hello', 'utf-8'); // => <Buffer 68 65 6c 6c 6f>
bin[0] = 0x48; //可更改

// slice方法不会返回一个新Buffer，而更像是返回了指向原Buffer中间的某个位置的指针
var bin = new Buffer([ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]);
var sub = bin.slice(2);
sub[0] = 0x65;
console.log(bin); // => <Buffer 68 65 65 6c 6f>

// 拷贝Buffer
var bin = new Buffer([ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]);
var dup = new Buffer(bin.length);
bin.copy(dup);
dup[0] = 0x48;
console.log(bin); // => <Buffer 68 65 6c 6c 6f>
console.log(dup); // => <Buffer 48 65 65 6c 6f>
```

#### Stream（数据流）
当内存中无法一次装下需要处理的数据，或者一边读取一边处理更加高效时，我们就需要使用数据流。NodeJS中通过各种`Stream`来提供对数据流的操作。`Stream`基于事件机制工作，所有`Stream`的实例都继承于NodeJS提供的[EventEmitter](http://nodejs.org/api/events.html)。

```javascript
var rs = fs.createReadStream(src);
var ws = fs.createWriteStream(dst);
rs.on('data', function (chunk) {
    if (ws.write(chunk) === false) {
        rs.pause();
    }
});
rs.on('end', function () {
    ws.end();
});
// 根据drain事件来判断什么时候只写数据流已经将缓存中的数据写入目标
ws.on('drain', function () {
    rs.resume();
});
```

#### File System（文件系统）
`fs`模块提供的API基本上可以分为以下三类：

- 文件属性读写。<br />其中常用的有`fs.stat`、`fs.chmod`、`fs.chown`等等。<br />
- 文件内容读写。<br />其中常用的有`fs.readFile`、`fs.readdir`、`fs.writeFile`、`fs.mkdir`等等。<br />
- 底层文件操作。<br />其中常用的有`fs.open`、`fs.read`、`fs.write`、`fs.close`等等。

基本上所有fs模块API的回调参数都有两个：

1. 第一个参数在有错误发生时等于异常对象
2. 第二个参数始终用于返回API方法执行结果

fs模块所有 API 都有对应的同步版本，用于无法使用异步操作，或者图标操作跟为方便的情况下使用，主要区别是：

- 同步API除了方法名的末尾多了一个`Sync`
- 异常对象与执行结果的传递方式不一样
```javascript
try {
    var data = fs.readFileSync(pathname);
    // Deal with data.
} catch (err) {
    // Deal with error.
}
```

#### Path（路径）
NodeJS提供了`path`内置模块来简化路径相关操作，并提升代码可读性:
```javascript
var cache = {};
  function store(key, value) {
    // 将传入的路径转换为标准路径
      cache[path.normalize(key)] = value;
  }
  store('foo/bar', 1);
  store('foo//baz//../bar', 2);
  console.log(cache);  // => { "foo/bar": 2 }

// 将传入的多个路径拼接为标准路径
path.join('foo/', 'baz/', '../bar'); // => "foo/bar"
// 获取扩展名
 path.extname('foo/bar.js'); // => ".js"
```

## 四、网络操作
NodeJS内置了 `http` 模块，可以让我们实现一个简单的HTTP服务器:
```javascript
const http = require("http");
http.createServer((request, response) => {
    response.writeHead(200, { "Content-Type": "text-plain" });
    response.end("Hello World
");
  }).listen(8124);
```

### API

#### HTTP
`http` 模块提供了两种使用方式：

- 作为服务端时，创建一个HTTP服务器，监听HTTP客户端请求并返回响应
- 作为客户端时，发起一个HTTP客户端请求，获取服务端响应

HTTP请求本质上是一个数据流，由请求头和请求体组成。HTTP请求在发送给服务器时，可以认为是按照从头到尾的顺序一个字节一个字节地以数据流方式发送的。而http模块创建的HTTP服务器在接收到完整的请求头后，就会调用回调函数，在回调函数中，除了可以使用request对象访问请求头数据外，还能把request对象当作一个只读数据流来访问请求体数据。

