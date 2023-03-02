
## 

## ![DSC_6552.jpg](https://cdn.nlark.com/yuque/0/2019/jpeg/296173/1570518303542-766dd04a-454d-4546-bf65-11b7784d7caa.jpeg#align=left&display=inline&height=4000&name=DSC_6552.jpg&originHeight=4000&originWidth=6000&size=3124975&status=done&width=6000)

## HTML5 WebSocket概述

作为新一代的web标准，HTML5为我们提供了很多有用的东西，比如canvas，本地存储（已经分离出去了），多媒体编程接口，当然还有我们的WebSocket。WebSocket是HTML5开始提供的一种浏览器与服务器间进行全双工通讯（full-duplex）的网络技术，可以传输基于信息的文本和二进制的数据。它于2011年被IETF定为标准 RFC 6455，同时WebSocket API也被W3C定为标准。


## 一、WebSocket产生的背景


### 1.黎明前的黑暗——实时web应用的需求

web应用的信息交互过程我想大家或多或少都知道一些，通常是客户端通过浏览器发出一个请求，然后服务器端在接受和审核请求后,进行处理并将结果返回给客户端，最后由客户端的浏览器将信息呈现出来。这种通信机制在信息交互不是特别频繁的情况下并没有太大的问题，但对于那些实时性要求高、海量数据并发的应用来说，就显得捉襟见肘了，比如现在常见的网页游戏，证券网站，RSS订阅推送，网页实时对话，打车软件等。通常当客户端准备呈现一些信息时，这些信息在服务器端很有可能就已经过时了。为了满足以上那些场景，大佬们研究出来了一些折衷方案，其中最常用的就是普通轮询和Comet技术，而Comet技术实际上就是轮询的改进，细分起来Comet有两种实现方式：

- 长轮询机制
- 流技术机制


#### 1.1 长轮询机制

长轮序是对普通轮询的改进和提高。普通轮询简单来说，就是客户端每隔一定的时间就向服务器端发送请求，从而以频繁请求的方式来保持客户端和服务器端的同步。这种同步方案的最大问题是，客户端已固定的频率发送请求时，很可能服务端的数据没有更新，产生很多无用的网络传输，非常低效。

为了减少无效的网络传输，长轮询对普通轮询进行了改进和提高，当服务器端没有数据更新时，链接会保持一段时间的周期，直到数据或状态发生改变或连接时间过期，通过这种机制我们就可以减少很多无效的客户端和服务器间的交互。当然，如果服务器端的数据变更非常频繁的话，这种机制并没有有效的提高性能，和普通轮询没有太大的区别，且长轮询也会耗费更多的资源，比如CPU,内存,带宽等。


#### 1.2 流技术机制

流技术机制简单来说就是客户端的页面使用一个隐藏的窗口向服务端发出一个长连接的请求。服务器接到请求后作出回应，并不断更新状态，以保证客户端和服务器端的连接不过期。通过这种机制就可以将服务器端的信息不断传向客户端，从而保证信息的时效性。但这种机制对于用户体验并不友好，需要针对不同的浏览器升级不同的方案来改进用户体验，同时这种机制如果在并发情况下发生时，会对服务器的资源造成很大压力。


### 2.黎明的到来——WebSocket

正是出于以上几种解决方案都有着各自的局限性,HTML5 WebSocket也就应运而生了，浏览器可以通过JavaScript借助现有的HTTP协议来向服务器发出WebSocket连接的请求，当连接建立后，客户端和服务器端就可以直接通过TCP连接来直接进行数据交换。这是由于websocket协议本质上就是一个TCP连接，所以在数据传输的稳定性和传输量上有所保证，且相对于以往的轮询和Comet技术在性能方面也有了长足的进步：<br />![](https://cdn.nlark.com/yuque/0/2019/jpeg/296173/1570518157644-8f16ce97-1e15-43dd-9724-fd16c9fb8caa.jpeg#align=left&display=inline&height=360&originHeight=360&originWidth=504&size=0&status=done&width=504)

有一点需要注意的是虽然websocket在通信时需要借助HTTP，但它本质上和HTTP有着很大的区别：

- WebSocket是一种双向通信协议，在建立连接之后，WebSocket服务端和客户端都能主动向对方发送或者接受数据。
- WebSocket需要先连接，只有再连接后才能进行相互通信。

他们的关系其实就和这张图表现的一样，虽然有相交的部分，但依然有着很大的区别：

![](https://cdn.nlark.com/yuque/0/2019/jpeg/296173/1570518157613-df69802f-fc65-42a6-a827-f45f3e07f16c.jpeg#align=left&display=inline&height=133&originHeight=133&originWidth=374&size=0&status=done&width=374)


## 二、WebSocket API的用法

由于每个服务器端的语言都有着自己的API，因此首先我们来讨论客户端的API：

```javascript
// 创建一个socket实例：
const socket = new WebSocket(ws://localhost:9093')
// 打开socket
socket.onopen = (event) => {
    // 发送一个初始化消息
  	socket.send('Hello Server!')
  	 // 服务器有响应数据触发
    socket.onmessage = (event) => { 
        console.log('Client received a message',event)
    }
    // 出错时触发，并且会关闭连接。这时可以根据错误信息进行按需处理
    socket.onerror = (event) => {
  	    console.log('error')
    }
    // 监听Socket的关闭
    socket.onclose = (event) => { 
        console.log('Client notified socket has closed',event)
    }
    // 关闭Socket
    socket.close(1000, 'closing normally') 
 }
```

是不是感觉HTML5 websocket所提供的API贼鸡儿简单，没错，就是这么简单。但有几点我们需要注意：

- 在创建socket实例的时候，new WebSocket()接受两个参数，第一个参数是ws或wss,第二个参数可以选填自定义协议，如果是多协议，可以是数组的方式。
- WebSocket中的send方法不是任何数据都能发送的，现在只能发送三类数据，包括UTF-8的string类型（会默认转化为USVString），ArrayBuffer和Blob，且只有在建立连接后才能使用。（感谢大佬指出错误，已修改）
- 在使用socket.close(code,[reason])关闭连接时，code和reason都是选填的。code是一个数字值表示关闭连接的状态号，表示连接被关闭的原因。如果这个参数没有被指定，默认的取值是1000 （表示正常连接关闭）,而reason是一个可读的字符串，表示连接被关闭的原因。这个字符串必须是不长于123字节的UTF-8 文本。


### 1.ws和wss

我们在上面提到过，创建一个socket实例时可以选填ws和wss来进行通信协议的确定。他们两个其实很像HTTP和HTTPS之间的关系。其中ws表示纯文本通信，而wss表示使用加密信道通信（TCP+TLS）。那为啥不直接使用HTTP而要自定义通信协议呢？这就要从WebSocket的目的说起来，WebSocket的主要功能就是为了给浏览器中的应用与服务器端提供优化的，双向的通信机制，但这不代表WebScoket只能局限于此，它当然还能够用于其他的场景，这就需要他可以通过非HTTP协议来进行数据交换，因此WebSocket也就采用了自定义URI模式，以确保就算没有HTTP，也能进行数据交换。

ws和wss：

- **ws协议**：普通请求，占用与HTTP相同的80端口
- **wss协议**：基于SSL的安全传输，占用与TLS相同的443端口。

注：有些HTTP中间设备有时候可能会不理解WebSocket，而导致各种诸如：盲目连接升级，乱修改内容等问题。而WSS就很好的解决了这个问题，它建立了一个端到端的安全通道，这个通道对中间设备模糊了数据，因此中间设备就不能感知到数据，也就无法对请求做一些特殊处理了。


## 三、WebSocket协议的规范

以下是一个典型的WebSocket发起请求到响应请求的示例：

```http
客户端到服务端：
GET / HTTP/1.1
Connection:Upgrade
Host:127.0.0.1:8088
Origin:null
Sec-WebSocket-Extensions:x-webkit-deflate-frame
Sec-WebSocket-Key:puVOuWb7rel6z2AVZBKnfw==
Sec-WebSocket-Version:13
Upgrade:websocket

服务端到客户端：
HTTP/1.1 101 Switching Protocols
Connection:Upgrade
Server:beetle websocket server
Upgrade:WebSocket
date: Thu, 10 May 2018 07:32:25 GMT
Access-Control-Allow-Credentials:true
Access-Control-Allow-Headers:content-type
Sec-WebSocket-Accept:FCKgUr8c7OsDsLFeJTWrJw6WO8Q=
```

我们可以看到，WebSocket协议和HTTP协议乍看并没有太大的区别，但细看下来，区别还是有些的，这其实是一个握手的http请求，首先请求和响应的，”Upgrade:WebSocket”表示请求的目的就是要将客户端和服务器端的通讯协议从 HTTP 协议升级到 WebSocket协议。从客户端到服务器端请求的信息里包含有”Sec-WebSocket-Extensions”、“Sec-WebSocket-Key”这样的头信息。这是客户端浏览器需要向服务器端提供的握手信息，服务器端解析这些头信息，并在握手的过程中依据这些信息生成一个28位的安全密钥并返回给客户端，以表明服务器端获取了客户端的请求，同意创建 WebSocket 连接。

当握手成功后，这个时候TCP连接就已经建立了，客户端与服务端就能够直接通过WebSocket直接进行数据传递。不过服务端还需要判断一次数据请求是什么时候开始的和什么时候是请求的结束的。在WebSocket中，由于浏览端和服务端已经打好招呼，如我发送的内容为utf-8 编码，如果我发送0x00,表示包的开始，如果发送了0xFF，就表示包的结束了。这就解决了黏包的问题。


## 四、兼容性情况

```
浏览器	                 支持情况
Chrome	            Supported in version 4+
Firefox	            Supported in version 4+
Internet Explorer	Supported in version 10+
Opera	            Supported in version 10+
Safari	            Supported in version 5+
```


## 五、Socket.IO

简单来说Socket.IO就是对WebSocket的封装，并且实现了WebSocket的服务端代码。Socket.IO将WebSocket和轮询（Polling）机制以及其它的实时通信方式封装成了通用的接口，并且在服务端实现了这些实时机制的相应代码。也就是说，WebSocket仅仅是Socket.IO实现实时通信的一个子集。Socket.IO简化了WebSocket API，统一了返回传输的API。传输种类包括：

- WebSocket
- Flash Socket
- AJAX long-polling
- AJAX multipart streaming
- IFrame
- JSONP polling。

我们来看一下服务端的Socket.IO基本API：

```javascript
// 引入socke.io
const io = require('socket.io')(80)
// 监听客户端连接,回调函数会传递本次连接的socket
io.on('connection',function(socket))
// 给所有客户端广播消息
io.sockets.emit('String',data)
// 给指定的客户端发送消息
io.sockets.socket(socketid).emit('String', data)
// 监听客户端发送的信息
socket.on('String',function(data))
// 给该socket的客户端发送消息
socket.emit('String', data)
```

另外，Socket.IO还提供了一个Node.JS API，它看起来很像客户端API。所以我们来看看它的实际应用吧：

```javascript
// socket-server.js

// 需要使用HTTP模块来启动服务器和Socket.IO
const http= require('http'), 
const io= require('socket.io')

const server= http.createServer(function(req, res){ 
    // 发送HTML的headers和message
    res.writeHead(200,{ 'Content-Type': 'text/html' })
    res.end('<p>Hello Socket.IO!<p>')
}); 
// 在8080端口启动服务器
server.listen(8080)

// 创建一个Socket.IO实例，并把它传递给服务器
const socket= io.listen(server)

// 添加一个连接监听器
socket.on('connection', function(client) { 

// 连接成功，开始监听
client.on('message',function(event){ 
    console.log('Received message from client!',event)
})
// 连接失败
client.on('disconnect',function(){ 
    clearInterval(interval)
    console.log('Server has disconnected')
  })
})
```

然后我们就可以启动这个文件了：

```
node socket-server.js
```

然后我们就可以创建一个每秒钟发送消息到客户端的发送器了；

```javascript
var interval= setInterval(function() { 
  client.send('This is a message from the server,hello world' + new Date().getTime()); 
},1000);
```

---

注：需要注意的是，如果我们想在前端使用socket.IO,我们需要下载这个：

```javascript
npm install socket.io-client --save
```

然后再连接网络：

```javascript
import io from 'socket.io-client'
const socket = io('ws://localhost:8080')
```

