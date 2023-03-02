
## 一、引擎，运行时，调用栈
JS引擎包括两部分：

- 内存堆
- 调用栈

堆栈溢出：当达到最大调用栈的时候发生


## 二、如何在 V8 引擎中书写最优代码的小技巧

- 对象属性的顺序：总是以相同的顺序实例化对象属性，这样隐藏类及之后的优化代码都可以被共享
- 动态属性：实例化之后为对象添加属性会致使为之前隐藏类优化的任何方法变慢
- 方法：重复执行相同方法的代码会比每次运行不同的方法更快（内联缓存）
- 数组：避免使用键不是递增数字的稀疏数组，稀疏数组即没有包含每个元素在其内部的一个哈希表。访问该数组中的元素也会更耗时。同时，试着避免预先分配大数组，最好是随着使用进行递增，也尽量减少删除数组中的元素
- 标记值：V8 用32位来表示对象和数字。超过31位，会进行装箱数字，将其转为浮点数并创建一个新的对象来存储这个数字。




## 三、内存管理及内存泄漏

### 1. 什么是内存
硬件层面：计算机内存有大量的 filp flops 组成，每个FF 包含少莲晶体管并能存储一个比特位。单个FF可以通过一个唯一标识符寻址，所以可以读写它。<br />内存中存储了很多东西：

- 全部程序使用的全部变量以及其他数据
- 程序的代码（包括操作系统的）

内存的实际分配是由编译器和操作系统一起完成的，编译器计算当前代码需要多少内存，然后交由操作系统去进行内存分配

### 2.内存生命周期

1. 内存分配：底层语言，开发者可以进行分配，而高级语言通常是操作系统帮忙分配的
2. 内存使用：程序实际使用之前所分配的内存的阶段
3. 内存释放：释放不再使用的整块内存，和内存分配一个道理，底层语言和高级语言不一样

### 3.动态内存分配
上面所提到的编辑器通过语法分析进行内存分配，实际情况下，有时候在编译时，我们会无法确定一个变量是否需要多少内存。因此就不能在栈中为变量分配内存空间，而是需要显式的从操作系统分配到正确的内存空间。

### 4.JavaScript中的内存管理

- 内存分配：完全交予操作系统
- 内存使用：js的内存使用分配的内存只要指的是内存读写，可以通过变量或者对象属性赋值，亦或是为函数传参使用内存
- 内存引用：在内存管理上下文中，如果对象 A 访问了另一个对象 B 表示 A 引用了对象 B。
- 内存垃圾回收：引用计数和标记清除。

引用计数无法解决循环引用的问题：
```rust
function f() {
  var o1 = {};
  var o2 = {};
  o1.P = O2; // O1 引用 o2
  o2.p = o1; // o2 引用 o1. 这就造成循环引用
}

f();
```
标记清除包含三个步骤：

1. 根：一般来说，根是代码中引用的全局变量。就拿JavaScript来说，window对象即是可看做根的全局变量。Node中对应的变量是 global, 垃圾回收期会构建出一份所有根变量的完整列表。
2. 算法会检测所有的根变量及他们的后代变量并标记它们为激活状态，任何根变量达不到的变量，会被标记为内存垃圾
3. 垃圾回收器会释放所有非激活状态的内存片段然后返回给操作系统

内存垃圾回收器的缺点则是具有不可预见性，我们无法确定的知道垃圾回收确切时机，因此，在某些情况下，陈程序会使用比时机需要更多的内存，而大多数的GC的实现是共享一种模式，即在内存分配期间进行垃圾回收。如果没有进行垃圾回收，大多数的垃圾回收器会保持闲置状态。

常见的内存泄漏：

1. 全局变量：js中可以使用this来创建全局变量，这可能会导致其挂在 全局根上。
2. 定时器以及被遗忘的回调函数
3. 闭包
4. 移出 DOM 引用


### 5.内存管理优化方案
内存管理的心得：

- 减少 GC 次数
- 减少内存占用

具体方案：

1. 使用对象池,zhe：
> 对象池**（英语：object pool pattern）是一种[设计模式](https://zh.wikipedia.org/wiki/%E5%AF%B9%E8%B1%A1%E6%B1%A0%E6%A8%A1%E5%BC%8F)。**一个对象池包含一组已经初始化过且可以使用的对象，而可以在有需求时创建和销毁对象。池的用户可以从池子中取得对象，对其进行操作处理，并在不需要时归还给池子而非直接销毁它。这是一种特殊的工厂对象。
> 若初始化、实例化的代价高，且有需求需要经常实例化，但每次实例化的数量较少的情况下，使用对象池可以获得显著的效能提升。从池子中取得对象的时间是可预测的，但新建一个实例所需的时间是不确定。

主要是有几个要点：

1. 按需创建
2. 预创建对象
3. 定时释放

其他：

- 减少对象的创建
- 对不使用的对象手动null
- 使用 Weakmap
- 生产环境不用 console 来包裹大对象，英雌 console的对象不会被垃圾回收
- 合理设计页面，按需使用对象、渲染页面、加载图片
- ImageData 对象是 JS 内存杀手，避免重复创建 ImageData 对象
- 重复使用 ArrayBuffer 
- 压缩图片、按需加载图片、按需渲染图片、使用尺寸恰当的图片、图片格式合理

图片优化

- 加载图片: 加载图片二进制格式到内存中并缓存，消耗100kb
- 解码图片: 二进制解码为像素格式
- 渲染图片: 通过CPU 或者 GPU 渲染图片

因此可以如此优化：

1. 使用 CSS3、SVG、IconFont、Canvas 替代图片。展示大量图片的页面，建议使用 Canvas 渲染而非直接使用img标签。
2. 适当压缩图片，可减小带宽消耗及图片内存占用。
3. 使用恰当的图片尺寸，即响应式图片，为不同终端输出不同尺寸图片，勿使用原图缩小代替 ICON 等，比如一些图片服务如 OSS。
4. 使用恰当的图片格式，如使用WebP格式等。详细图片格式对比，使用场景等建议查看[web前端图片极限优化策略](http://jixianqianduan.com/frontend-weboptimize/2015/11/17/front-end-image-optmize.html)。
5. 按需加载及按需渲染图片。
6. 预加载图片时，切记要将 img 对象赋为 null，否则会导致图片内存无法释放。当实际渲染图片时，浏览器会从缓存中再次读取。
7. 将离屏 img 对象赋为 null，src 赋为 null，督促浏览器及时回收内存及像素格式内存。
8. 将非可视区域图片移除，需要时再次渲染。


## 四、事件循环 和 async await
setTimeout 工作原理：需要注意的是 setTimeout 并没有自动把回调添加到事件循环队列，他创建了一个定时器，当定时器过期时，宿主环境会把回调函数添加到事件循环队列中。


promise小 case: 我们可以通过new Promise 来创建 Promise, 那么我们也应当使用 p instanceof Promise 来检测某个对象是否是 promise类的实例。但其实不太行，应为我们可以从另一个浏览器窗口（比如iframe）获取 promise 实例，iframe 中的 promise 不同于当前窗口的promise，这会使其识别promise 失败

书写好的异步代码的五个小技巧：

1. 简洁：使用async await
2. 错误处理： async await 可直接使用 try catch 来捕获错误
3. 条件语句
4. 堆栈：和 async/await 不同的是，从链式 promise 返回的错误堆栈中无法得知发生错误的地方
5. 调试：如果使用 promise，你就会明白调试它们是一场噩梦。例如，如果你在 .then 代码块中设置一个断点并且使用诸如 "stop-over" 的调试快捷键，调试器不会移动到下一个 .then 代码块，因为调试器只会步进同步代码。使用 async/await 你可以就像一般的同步函数那样逐句通过 await 调用。


## 五、深入理解ws，和带有SSE机制的HTTP2
帧数据：数据可拆分为多个帧，第一帧所传输的数据里面包含的一个操作码表示传输值得数据类型。当我们传输二进制时，它会以浏览器置顶的Blob来表示。

数据分片：有效载荷数据可以被分为多个独立帧，接收端会缓冲这些知道fin位有值。<br />连接帧的逻辑：

- 接收第一帧
- 记住操作码
- 连接帧有效载荷直到 fin 位有值
- 断言每个包的操作码都为 0 

ws 和 http2 sse的优缺点

- sse基于HTTP，所以可以使用http2的一些特性，比如多路复用流。
> 流：即是一个HTTP2连接中，在客户端和服务端进行交换传输的一个独立的双向帧序列。他的主要特点之一即单个HTTP2连接可以包含多个并发打开的流，在每一终端交错传输来自多个多个流的帧。


## 六、WebAssembly 
WebAssembly是一种用于开发网络应用的高效，底层的字节码。

- 加载时间： 浏览器会更快的加载 wasm， 因为 wasm 知会传输已经编译好的 wasm 文件，而wasm 是底层的类汇编语言，具有非常简洁的二进制格式。
- 执行速度：比原生JavaScript慢20%，但它的安全性非常好，它是一种格式，会被编译进沙箱环境中且在大量的约束条件下运行以保证任何安全漏洞或者使之强化以对抗漏洞。

wasm的使用场景：

- 解决大量计算密集型的计算
- 处理计算密集型的库


## 七、Web Workers 分类及使用场景
异步编程的局限性：异步编程只能解决一部分单线程JavaScript的单线程限制，但如果遇到CPU密集的任务，还是不能解决很多问题。

Web Workers 运行我们运行处理 CPU 计算密集型任务的耗时脚本而不会阻塞UI。值得注意的是 JS 并没有定义线程模型。web workers 并不是JS 的一部分，这是有浏览器所提供的（也就是运行环境）。在Node 中就没有 Web Workers 。它有 cluster。规范中有三种类型的 Web  Workers:

- [Dedicated Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers): 是由主进程实例化并且只能与之进行通信
- [Shared Workers](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker)：可以被运行在同源的所有进程访问
- [Service workers](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorker_API)：是一个事件驱动的worker，它由源和路径组成，它可以控制其关联的网络，拦截及修改导航资源的请求。

Web worker 运行原理：它运行与浏览器的一个隔离线程中，因此它们所执行的代码必须被包含在一个单独的文件之中。

通信方式：

- postMessage
- 广播通信
```rust
// 连接到一个广播信道
var bc = new BroadcastChannel('test_channel');
// 发送简单信息示例
bc.postMessage('This is a test message.');
// 一个在控制台打印消息的简单事件处理程序示例
// logs the message to the console
bc.onmessage = function (e) { 
  console.log(e.data); 
}
// 关闭信道
bc.close()

```
                          <br />                    ![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1631603000108-50a19933-42cf-4f08-a8a2-b411a23bca01.png#clientId=udfaf4443-9c5f-4&from=paste&height=256&id=u0bff88ed&name=image.png&originHeight=768&originWidth=1024&originalType=binary&ratio=1&size=69142&status=done&style=none&taskId=u7d13d800-69b3-4f17-a7da-fc94be4eaa9&width=341.3333333333333)

有两种向 Web Workers 发送消息的方法：

- 复制消息：消息被序列化，复制，然后发送出去，接着在接收端反序列化。页面和 worker 没有共享一个相同的消息实例，所以在每次传递消息过程中最后的结果都是复制的。大多数浏览器是通过在任何一端自动进行 JSON 编码/解码消息值来实现这一功能。正如所预料的那样，这些对于数据的操作显著增加了消息传送的性能开销。消息越大，传送的时间越长。
- 消息传输：这意味着最初的消息发送者一发送即不再使用（）。数据传输非常的快。唯一的限制即只能传输 [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) 数据对象。

<br />web workers 能用的：

- navigator对象
- location 对象
- XMLhttpRequest
- setTimeout
- Application Catch
- 使用 importScripts 来引用外部脚本
- 创建其他 web workers

不能的：

- DOM
- window
- document
- parent对象

web workers 的最佳使用场景

- 射线追踪
- 加密
- 预取数据
- 渐进式网络应用
- 拼写检查


## 八、Service Worker生命周期及使用场景
service worker 大体上是一种 Web worker，更准确的说，像是一种 Shared worker。主要应用于渐进式网络，所谓的渐进式网络的主要需求是可以在网络不稳定或者无网的情况下使用。

- service woker 运行爱全局脚本上下文中
- 不和特定网页相关联
- 不能够访问 Dom

Service woker的生命周期和网页完全不同，它由几个步骤组成

1. 下载：这发生于浏览器下载包含 Service Worker 代码的.js文件
2. 安装：在完成 service woker注册时，它会让浏览器在后台开始安装 service worker 的步骤。
```typescript
// 检查是否支持
if ('serviceWorker' in navigator) {
  window.addEventListener('load', function() {
    navigator.serviceWorker.register('/sw.js').then(function(registration) {
      // 注册成功
      console.log('ServiceWorker registration successful');
    }, function(err) {
      // 注册失败
      console.log('ServiceWorker registration failed: ', err);
    });
  });
}

```

3. 激活：一旦激活，Service Worker 就可以开始控制在其作用域内的所有页面。注册了 Service Worker 的页面直到再次加载的时候才会被 Service Worker 控制。当 Service Worker 开始进行控制，它有以下几种状态：
- 处理来自页面的网络或者消息请求所触发的 fetch 及 message 事件
- 中止以节约内存

生命周期如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1634096303398-439f7e18-d8bb-42d2-8068-54c19e6bc3e3.png#clientId=ua2cc324f-e115-4&from=paste&height=186&id=u979e204d&name=image.png&originHeight=558&originWidth=854&originalType=binary&ratio=1&size=42819&status=done&style=none&taskId=uc863433b-e043-4d59-a80a-0632580b710&width=284.6666666666667)

## 十、使用 MutationObserver 检测 DOM 变化

主要用来检测 DOM 变化的网页接口，可以在以下场景下使用：

- 通知要不关乎当前所在的页面所发生的一些变化
- 在所见即所得的编辑器很重，使用这个可以收起任意时间点上的更改，从而轻松时间撤销和重做功能

使用：
```typescript
var mutationObserver = new MutationObserver(function(mutations) {
  mutations.forEach(function(mutation) {
    console.log(mutation);
  });
});
```
创建的实例对象拥有三个方法：

- observe－开始进行监听 DOM 更改。接收两个参数－要观察的 DOM 节点以及一个配置对象。
- disconnect－停止监听变化。
- takeRecords－触发回调前返回最新的批量 DOM 更改。
```typescript
// 开始监听页面根元素 HTML 变化。
mutationObserver.observe(document.documentElement, {
  attributes: true,
  characterData: true,
  childList: true,
  subtree: true,
  attributeOldValue: true,
  characterDataOldValue: true
});
```

## 十二、网络层，性能和安全
  


