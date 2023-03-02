
### 一、HTTP缓存概述（HTTP Cache）

要搞清楚HTTP缓存，首先当然是要搞清楚缓存是啥，按照MDN的描述，缓存是这样的：

> 缓存是一种保存资源副本并在下次请求时直接使用该副本的技术。当 web 缓存发现请求的资源已经被存储，它会拦截请求，返回该资源的拷贝，而不会去源服务器重新下载。这样带来的好处有：缓解服务器端压力，提升性能(获取资源的耗时更短了)。对于网站来说，缓存是达到高性能的重要组成部分。缓存需要合理配置，因为并不是所有资源都是永久不变的：重要的是对一个资源的缓存应截止到其下一次发生改变（即不能缓存过期的资源）。
> 上面已经将会缓存是什么描述的很清楚了，而HTTP缓存顾名思义，就是通过HTTP协议，来实现对资源缓存的目的。总的来说，HTTP缓存主要通过两个HTTP头来实现的，其中Expires是由HTTP1.0提供的，而Cache-Control则是由HTTP1.1所提供的：

![](https://cdn.nlark.com/yuque/0/2019/png/296173/1576766614906-ba19e347-1af0-4b2f-bb01-3ea2c26c5b19.png#align=left&display=inline&height=321&originHeight=321&originWidth=372&size=0&status=done&style=none&width=372)<br />下面我们就来对这两个头进行一个了解。


### 二、Expires

Expires是由HTTP1.0所提供的支持HTTP缓存的头部，由服务器返回，用GMT格式的字符串表示：

```http
expires: Tue, 14 Aug 2018 14:32:49 GMT
```

而读取缓存的条件则是：缓存过期时间（服务器的时间）< 当前时间（客户端的时间）<br />值得注意的是，我们在expires所设置的时间是一个绝对的时间，而且所参照的是用户电脑上所设置的时间。这种绝对的时间很容易出问题，当用户本地的时间不准确，或用户进行跨时区的移动时，这个时间很可能就会过期，而无法发挥它本应该发挥的作用。<br />其次，在HTTP1.0里，没有提供相应的配置缓存的方法，只是提供了这个强缓存的头部而已，不足以满足项目对缓存多样化的需求。也正是出于以上两个原因，在HTTP1.1中对HTTP缓存又进行了升级。


### 三、 Cache-Control

正是由于Expires存在着很多不足，所以HTTP1.1又为我们提供了<br />Cache-Control主要可配置的参数有以下几个：

1. max-age 会指定从请求的时间开始，允许获取的响应被重用的最长时间（单位：秒）。例如，“max-age=60”表示可在接下来的 60 秒缓存和重用响应。
2. Public 表示响应可被任何缓存区缓存
3. Private 表示对于单个用户的整个或部分响应消息，不能被共享缓存处理。这些响应通常只为单个用户缓存，因此不允许任何中间缓存对其进行缓存。例如，用户的浏览器可以缓存包含用户私人信息的 HTML 网页，但 CDN 却不能缓存。
4. no-cache 表示必须先与服务器确认返回的响应是否发生了变化，然后才能使用该响应来满足后续对同一网址的请求。因此，如果存在合适的验证，no-cache 会发起往返通信来验证缓存的响应，但如果资源未发生变化，则可避免下载。
5. no-store则要简单得多。它直接禁止浏览器以及所有中间缓存存储任何版本的返回响应，例如，包含个人隐私数据或银行业务数据的响应。每次用户请求该资产时，都会向服务器发送请求，并下载完整的响应。
6. min-fresh 表示客户端可以接收响应时间小于当前时间加上指定时间的响应。（用的不多）
7. max-stale 表示客户端可以接收超出超时期间的响应消息。如果指定max-stale消息的值，那么客户机可以接收超出超时期指定值之内的响应消息。（用的不多）


### 四、强缓存

上面已经将Catch-Control做了一个简单的介绍，而具体使用它们二者进行缓存操作的具体实现又分为强缓存与协商缓存。首先来聊一聊强缓存。<br />强缓存是利用Expires或者Cache-Control这两个http response header实现的，它们都用来表示资源在客户端缓存的有效期。在这个有效期内当浏览器对某个资源的请求命中了强缓存时，其返回的http状态为200，并且不会去对服务器进行请求，而是直接使用其本地的缓存。<br />具体实现如下：

```http
expires: Tue, 21 Aug 2018 10:17:45 GMT
cache-control: max-age=691200
```

只要存在以上两个头部信息的其中一个，我们就可以对资源进行强缓存了。另外需要注意的是，当Catch-Control的优先级是要高于expires的。<br />总的来说，强缓存是前端性能优化的一大助力。当我们页面存在很多长期不变的静态资源时，都应该对其进行强缓存的处理，我们通常可以为这些静态资源全部配置一个超时时间很长的Expires或Cache-Control。当用户在访问网页时，就只会在第一次加载时从服务器请求静态资源，在往后访问该页面时，就只要缓存没有失效并且用户没有强制刷新的条件下都会从自己的缓存中加载。这样既节省了资源加载的时间的消耗，又不会去访问服务器，可以有效地为服务器减压。<br />不过强缓存也存在一个很大的弊端，那就是对于动态资源它就有点力不从心了。因为如果我们对动态资源进行了强缓存，那么很可能会在这动态资源更改后，浏览器还是会直接去请求没有更改前的动态资源。也这是由于这方面的考虑，在强缓存外还存在着协商缓存的缓存方案。


### 五、协商缓存

当浏览器对某个资源的请求没有命中强缓存，就会发一个请求到服务器，验证协商缓存是否命中，如果协商缓存命中，请求响应返回的http状态为304并且会显示一个Not Modified的字符串；<br />若未命中请求，则将资源返回客户端，并更新本地缓存数据，并返回200的状态码。<br />除此之外，我们也可以设置为协商缓存，以解决动态资源缓存与更新的问题，首先我们来看一张关于协商缓存的图：<br />![](https://cdn.nlark.com/yuque/0/2019/png/296173/1576766614956-e323cc04-e164-42a2-9d82-8e3230995167.png#align=left&display=inline&height=588&originHeight=588&originWidth=1074&size=0&status=done&style=none&width=1074)<br />可以看到，在这个实现了协商缓存的Cache-Control中，设置了no-catch，当我们设置为no-catch时，我们就是可以直接去访问服务器，去查看该资源的更改情况，已确定是是否需要使用缓存。而在校验的这一步我们就需要使用以下几个头来帮助验证资源的更改情况了：

- Last-Modified：表示这个响应资源的最后修改时间。web服务器在响应请求时，会告诉浏览器该资源的最后修改时间，但它的最小单位是秒级，也就是如果我们在1秒内多次修改该资源，那么Last-Modified也无法发挥其应有的作用。
- If-Modified-Since：当资源过期时（强缓存失效），发现资源具有Last-Modified声明，则再次向web服务器请求时带上头 If-Modified-Since，表示请求时间。web服务器收到请求后发现有头If-Modified-Since 则与被请求资源的最后修改时间进行比对。若最后修改时间较新，说明资源又被改动过，则响应整片资源内容（写在响应消息包体内），HTTP 200；若最后修改时间较旧，说明资源无新修改，则响应HTTP 304 (无需包体，节省浏览)，告知浏览器继续使用所保存的cache。<br />也正是由于 Last-Modified存在着缺陷，我们就需要ETag来帮助我们来对资源的更改进行判断：
- Etag：当Web服务器响应请求时，会告诉浏览器当前资源在服务器的唯一标识（生成规则是由服务器决定的）。就比如在Apache中，ETag的值，其默认是对文件的索引节（INode），大小（Size）和最后修改时间（MTime）进行Hash后得到的。
- If-None-Match：当资源过期时，如果发现资源具有Etage声明，则再次向web服务器请求时带上头If-None-Match （Etag的值）。Web服务器收到请求后发现有头If-None-Match， 就会将其被请求的资源的相应校验字段进行对比，然后再决定返回200或304。<br />这就是强缓存与协商缓存的大部分情况了，具体的流程可见下图：<br />![](https://cdn.nlark.com/yuque/0/2019/png/296173/1576766614893-53e5d333-79d9-4c83-b373-5f8ac085ca9c.png#align=left&display=inline&height=650&originHeight=650&originWidth=1041&size=0&status=done&style=none&width=1041)<br />原图链接：[https://user-gold-cdn.xitu.io/2018/8/16/165411b79180df27?w=1041&h=650&f=png&s=85785](https://user-gold-cdn.xitu.io/2018/8/16/165411b79180df27?w=1041&h=650&f=png&s=85785)


#### 六、定义最佳 Cache-Control 策略

![](https://cdn.nlark.com/yuque/0/2019/png/296173/1576766614900-96c1439f-3f5f-421e-85ca-cc5a043f6f90.png#align=left&display=inline&height=600&originHeight=600&originWidth=595&size=0&status=done&style=none&width=595)<br />通常我们可以按照上面这张流程图来对HTTP缓存进行相应的配置，详情可以看这篇文章：
> [https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)?

这位大佬对缓存的配置做了一个很好的阐述。

参考资料：

- [https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)?
- [https://www.cnblogs.com/lyzg/p/5125934.html#_label3](https://www.cnblogs.com/lyzg/p/5125934.html#_label3)
- [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching_FAQ](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching_FAQ)
