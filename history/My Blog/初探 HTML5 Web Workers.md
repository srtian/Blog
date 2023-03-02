
### 前言

这段时间一这被很多事情所牵绕，没能好好的写博客学习，感觉很难受，但所幸事情大都告一段落了。对于 Web Workers 的学习主要是由于在上这周面试的时候，面试的一个大佬，问到了这个问题，而我只知道它有这个东西，但具体如何实现它，以及它的使用场景我不是很清楚。因此这两天花了些时间看了不少文章，来总结一下。



### 一、Web Workers是什么

> Web Worker为Web内容在后台线程中运行脚本提供了一种简单的方法。线程可以执行任务而不干扰用户界面。此外，他们可以使用XMLHttpRequest执行 I/O (尽管responseXML和通道属性总是为空)。一旦创建， 一个worker 可以将消息发送到创建它的JavaScript代码, 通过将消息发布到该代码指定的事件处理程序 (反之亦然)。 —— MDN


众所周知，JavaScript是单线程的编程语言，也就是说，当我们在页面中进行一个较为耗时的计算的JavaScript代码时，在这段代码执行完毕之前，页面是无法响应用户操作的。也正是出于这个原因，HTML5为我们提供了 Web Workers 以解决这种问题，当我们需要在JavaScript中来进行耗时的计算或诸如此类的问题时，我们可以使用 Web Workers 在浏览器的后台启动一个独里的 Worker 线程来专门负责这段代码的运行，而不会阻碍后面代码的运行。



### 二、Web Workers的使用



#### 1. 实例化一个 Worker

实例化运行一个 Worker 很简单，我们只需要 new 一个 Worker 全局对象即可。

```javascript
var worker = new Worker('./worker.js')
```

它接受一个 filepathname String 参数，用于指定 Worker 脚本文件的路径。然后我们就可以在 worker.js 中写下一些代码:

```javascript
console.log('my_WOEKER:', 'srtian')
```

另外，通过URL.createObjectURL()创建URL对象，也可以实现创建内嵌的worker:

```javascript
var myTask = `
    var i = 0;
    var timedCount = () => {
        i = i+1;
        postMessage(i);
        setTimeout(timedCount, 1000);
    }
    timedCount();
`;

var myblob = new Blob([myTask]);
var myWorker = new Worker(window.URL.createObjectURL(myblob));
```

> 需要注意的是，传入 Worker 构造函数的参数 URI 必须遵循同源策略。


此外因为Worker线程的创建的是异步的，所以主线程代码不会阻塞在这里等待 worker 线程去加载、执行指定的脚本文件，而是会立即向下继续执行后面代码这点也需要注意。



#### 2. 数据通信

当我们实例化一个 Worker 线程后，Worker不会相互，或者与主程序共享任何作用域或资源——那会将所有的多线程编程的噩梦带到我们面前——取而代之的是一种连接它们的基本事件消息机制。因此他们需要通过基于事件监听机制的message来进行通信，我们在new Worker()后悔返回一个实例对象，它包含了一个postMessage的方法，我们可以通过调用这个方法来给worker线程传递信息，我们也可以给这个对象监听事件，从而在worker线程中出发事件通信的时候能接收到数据。

```javascript
var worker = new worker('./worker.js')
worker.addEventListener('message', function(e) {
    console.log('worker receive:', e.data )
}
worker.postMessage('hello worker,this is main.js')
```

然后在worker.js这个脚本中，我们就可以调用全局函数postMessage和全局的onmessage赋值来发送和监听数据和事件了。

```javascript
// 监听事件
onmessage = function (e) {
  console.log('WORKER RECEIVE：', e.data);
  // 发送数据事件
  postMessage('Hello, this is worker.js');
}
```

需要注意的是 worker 支持 JavaScript 中所有类型的数据传递，可以传递一个 Object 数据；但这里的数据传递（主要是 Object 类型）并不是共享，而是复制。发送端的数据和接收端的数据是复制而来，并不指向同一个对象，此外这里的复制不是简单的拷贝，而是通过两端的序列化/解序列化来实现的，一般来说浏览器会通过 JSON 编码/解码；当然，这里的更多细节部分会由浏览器来处理，我们并不需要关心这些，只需要明白两端的数据是复制而来，互相独立的就行了。



#### 3. 错误处理机制

当 worker 出现运行中错误时，它的 onerror 事件处理函数会被调用。它会收到一个扩展了 ErrorEvent 接口的名为 error的事件。

该事件不会冒泡并且可以被取消；为了防止触发默认动作，worker 可以调用错误事件的 preventDefault() 方法。

错误事件有以下三个用户关心的字段：

- message: 可读性良好的错误消息。
- filename: 发生错误的脚本文件名。
- lineno: 发生错误时所在脚本文件的行号。

实际操作如下：

```javascript
var worker = new Worker('./worker.js');

// 监听消息事件
worker.addEventListener('message', function (e) {
  console.log('MAIN RECEIVE： ', e.data);
});
// 也可以使用 onMessage 来监听事件：


// 监听 error 事件
worker.addEventListener('error', function (e) {
  console.log('MAIN ERROR：', e);
  console.log('MAIN ERROR：', 'filename:' + e.filename + '---message:' + e.message + '---lineno:' + e.lineno);
});


// 触发事件，传递信息给 Worker
worker.postMessage({
  m: 'Hello Worker, this is main.js'
});
```



#### 4. 终止 Worker

当我们在不需要 Worker 继续运行时，我就需要终止掉这个线程，这时候我们就可以调用 worker 的 terminate 方法:

```javascript
worker.terminate()
```

worker 线程会被立即杀死，不会有任何机会让它完成自己的操作或清理工作。

而在worker线程中，workers 也可以调用自己的 close  方法进行关闭：

```
close()
```



### 三. Web Workers的兼容

由于Web Workers是HTML5所提供的，因此从兼容性上来说，还是需要注意的。总的兼容情况如下图所示：<br />
![](https://qiutc.me/img/section-webworker-1.png#alt=image)<br />
图片来源：[https://caniuse.com/#feat=webworkers](https://caniuse.com/#feat=webworkers)

我们可以看到，虽然web worker很不错，但如果我们的代码执行在较老的浏览器中时，是缺乏支持的。但由于worker是一个API而不是语法，因此我门还是可以去填补它的。

这一块的详情可以去看——《你不知道的JavaScript中卷》关于 web worker 的那一节。



### 四、Web Workers支持的JavaScript特性

由于在 Worker 线程的运行环境中没有 window 全局对象，也无法访问 DOM 对象，所以一般来说我们在这只能执行纯JavaScript的计算操作，当然1我们那：

- setTimeout()， clearTimeout()， setInterval()， clearInterval()：有了设计个函数，就可以在 Worker 线程中执行定时操作了；
- XMLHttpRequest 对象：意味着我们可以在 Worker 线程中执行 ajax 请求；
- navigator 对象：可以获取到 ppName，appVersion，platform，userAgent 等信息；
- location 对象（只读）：可以获取到有关当前 URL 的信息；
- 应用缓存
- 使用 importScripts() 引入外部 script
- 创建其他的 Web Worker



### 五、Web Worker 的实践

总的来说，Web Worker为我们带来了强大的计算能力，我们可以加载一个JavaScript进行大量的复杂计算，而用不挂起主进程。并通过postMessage，onmessage进行通信，这也解决了大量计算对UI渲染的阻塞问题。



#### 应用场景



##### 1、数学运算

Web Worker最简单的应用应该就是用来进行后台计算了，这对CPU密集型的场景再适合不过了。



##### 2、图像处理

通过使用从 canvas 中获取的数据，可以把图像分割成几个不同的区域并且把它们推送给并行的不同Workers来做计算，对图像进行像素级的处理，再把处理完成的图像数据返回给主页面。



##### 3、大数据的处理

目前mvvm框架越来越普及，基于数据驱动的开发模式也越愈发流行，未来大数据的处理也可能转向到前台，因此我们将大数据的处理交给在Web Worker也是很好的。



##### 4. 数据预处理

为优化的网站或 web 应用的数据加载时长，我们可以使用 Web Worker 预先获取一些数据，存储起来以备后续使用，因为它绝不会影响应用的 UI 体验。



##### 5. 大量的 ajax 请求或者网络服务轮询

由于在主线程中每启动一个XMLHttpRequest请求都会消耗资源，虽然在请求过程中浏览器另外开了一个线程，但是在交互过程中还是需要消耗主线程资源；而使用worker则不会过多占用主线程，只是启动worker过程时比较耗资源。

参考资料：

1. 《你不知道的JavaScript中卷》
2. [https://juejin.im/post/59c1b3645188250ea1502e46](https://juejin.im/post/59c1b3645188250ea1502e46)
3. [https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers)
4. [https://qiutc.me/post/the-multithread-in-javascript-web-worker.html](https://qiutc.me/post/the-multithread-in-javascript-web-worker.html)
5. [https://juejin.im/post/5a90233bf265da4e92683de3](https://juejin.im/post/5a90233bf265da4e92683de3)
