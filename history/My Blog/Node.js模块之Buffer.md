
### Buffer的简单介绍

由于在Node中，我们的应用需要处理网络协议、操作数据库、处理图片、接收上传文件等，在网络流和文件的操作中，还要处理大量二进制数据。而JavaScript原生方法，并不能满足这些需求，出于这些需求Buffer 也就应运而生了。<br />
Buffer是Node.js中非常重要的一个模块，<br />
Node.js的官方文档是这么介绍Buffer的：[Node.js_Buffer](https://nodejs.org/api/buffer.html#buffer_buffer) :

> Prior to the introduction of TypedArray, the JavaScript language had no mechanism for reading or manipulating streams of binary data. The Buffer class was introduced as part of the Node.js API to enable interaction with octet streams in TCP streams, file system operations, and other contexts.


> With TypedArray now available, the Buffer class implements the Uint8Array API in a manner that is more optimized and suitable for Node.js.


> Instances of the Buffer class are similar to arrays of integers but correspond to fixed-sized, raw memory allocations outside the V8 heap. The size of the Buffer is established when it is created and cannot be changed.


> The Buffer class is within the global scope, making it unlikely that one would need to ever use require('buffer').Buffer.


简单来说：

- Buffer 库就是为 Node.js 带来<br />
了一种存储原始数据的方法，从而可以让 Nodejs 处理二进制数据。
- 每当我们需要在 Nodejs 中可以使用 Buffer 来处理诸如TCP流，文件系统操作和其他上下文中的移动的数据。原始数据存储在 Buffer 类的实例中。
- Buffer 类似于一个整数数组，但它对应于 V8 堆内存之外的一块原始内存，且创建后无法更改。
- 由于 Buffer 是一个全局范围定义的，所以我们无需引用就可以使用。

值得注意的是Buffer是一个典型的JavaScript与 C++ 结合的模块。它将性能相关部分用 C++ 实现，将非性能相关的部分则用 JavaScript 实现。<br />
前面有提到 Buffer 所占的内存不是由V8分配的，而是独立于 V8 堆内存之外，通过 C++ 来实现内存申请、javascript 分配内存。每当我们使用Buffer.alloc(size)请求一个Buffer内存时，Buffer会以8KB为界限来判断分配的是大对象还是小对象，小对象存入剩余内存池，不够再申请一个8KB的内存池；大对象直接采用C++层面申请的内存。因此，对于一个大尺寸对象，申请一个大内存比申请众多小内存池快很多。



### Buffer的简单使用

```javascript
// 创建一个长度为 10、且用 20 填充的 Buffer。
const buf1 = Buffer.alloc(10, 20)
console.log(buf1)
// <Buffer 14 14 14 14 14 14 14 14 14 14>
```

Buffer同样也可以与string互相转化：

```javascript
const buf2 = Buffer.from('javascript')
console.log(buf2)
// <Buffer 6a 61 76 61 73 63 72 69 70 74>
```

需要注意的是，Buffer.from不支持传入数字：

```javascript
Buffer.from(12345)
// TypeError [ERR_INVALID_ARG_TYPE]: The "value" argument must not be of type number. Received type number at Function.from (buffer.js:215:11)
```

因此如果我们需要传入数字，可以将其以数组的形式传进去：

```javascript
const buf = Buffer.from([1, 2, 3, 4, 5]);
console.log(buf); //  <Buffer 01 02 03 04 05>
```

但是这种方式存在一个问题，当存入不同的数值的时候buffer中记录的二进制数据会相同，如下所示：

```javascript
const buf1 = Buffer.from([127, -1])
console.log(buf1)
// <Buffer 7f ff>

const buf2 = Buffer.from([127, 255])
console.log(buf2) 
// <Buffer 7f ff>

console.log(buf2.equals(buf1))  // true
```

当要记录的一组数全部落在0到255（readUInt8来读取）这个范围, 或者全部落在-128到127（readInt8来读取）这个范围那么就没有问题，否则的话就强烈不推荐使用Buffer.from来保存一组数。因为不同的数字读取时应该调用不同的方法。

参考文档：

- 《深入浅出Node.js》
- [https://juejin.im/post/5afd57e851882542ac7d76af](https://juejin.im/post/5afd57e851882542ac7d76af)
- [http://nodejs.cn/api/buffer.html#buffer_buffer](http://nodejs.cn/api/buffer.html#buffer_buffer)
