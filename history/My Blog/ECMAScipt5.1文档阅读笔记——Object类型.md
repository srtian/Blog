---


title: ECMAScript5.1文档阅读笔记——Object类型<br />
categories: JavaScript<br />
tags:

- JavaScript

---


### 前言

起床有点发烧，因此学些新东西有点力不从心了，所以想了想还是复习复习JavaScript吧。然后想着ES6虽然用的还蛮多的，但ES5才是本源啊，就找到了ECMAScipt5.1文档读了部分自己感兴趣的部分，还可以学学英语，做个笔记。。。

> [http://www.ecma-international.org/ecma-262/5.1/](http://www.ecma-international.org/ecma-262/5.1/)



### Object 类型

这部分大体上我在看高程的时候就已经知道了，不过这三种属性竟然都属于Object类型，而不是包含关系，这倒是让我涨姿势了。

。首先Object是一个属性的集合。这里说的属性可以是数据属性，也可以是访问器属性，也可以是内部属性。

- 数据属性（data property）
- 访问器属性：Vue的双向绑定就靠它
- 内部属性（internal property）：对于这个属性，我们不能直接通过JavaScript来操作，就比如[[prototype]],我们就不能直接操作，因此在对象中我们通过**proto**来访问它。


### Property Attributes

这部分主要是讲了数据属性和访问器属性，大致上好像和高程说的没什么两样，两者都是四个特性：


##### 数据属性：

- [[Value]]：值
- [[Writable]]：布尔类型，表示可写。如果设置为false，就无法更改value的值。
- [[Enumerable]]：布尔类型，表示可枚举。如果设置为false,就无法被for-in等枚举出来。
- [[Configurable]]：布尔类型，如果设置为false。就无法删除该属性或者改变该属性为访问器属性，也无法改变它的特性。


##### 访问器属性：

- [[Get]]：是一个Object or Undefined。如果是一个 Object 对象，那么它必须就是一个函数对象。用于获取值
- [[Set]]：同上，不同的是它用于设置值，Vue的数据劫持就靠这两个属性来实现的。
- [[Enumerable]]：同数据属性
- [[Configurable]]：同数据属性


#### 内部属性：

对于内部属性我认识的并不多，其一是由于所看的书好像都没有对内部属性有个完整的介绍，其二是由于内部属性都无法直接访问，因此用的少。不过按照文档的描述，这些内部属性倒不是 ECMAScript的一部分，知识为了说明他们而定义的，也就是说有这么一个内部实现在这里，但其实本质上他们并不是ECMAScript的一份子，只是为了描述这些内部实现而搞的东西而已。下面这是文档中的表8，适用于所有 ECMAScript 对象的内部属性：（还有一个表9，表示部分对象存在的内部属性，其文档对于这个表也没啥解说，因此我就不知道了。。。）<br />
![](https://images.gitee.com/uploads/images/2018/0808/163603_47686baa_1575229.png#alt=image)<br />
挑重要的说：

- [[Prototype]]：这玩意所有对象都有，可以说是最重要的一个内部属性了（我个人是这么认为的），因为在JavaScript中，对象都靠它来实现继承，也就是我们通常所说的原型。按照规范上的描述，所有的原型链必须是有限长度的，终点便是null。
- [[Class]]：这个 内部属性的值用于内部区分对象的种类。这东西就是Object.prototype.toString.call的原理所在，规范中只有Object.prototype.toString.call可以访问这个属性
- [[Get]]：返回命名属性的值
- [[Put]]：将指定的属性设置为第二个参数的值。

其他的对我来讲，用的都不多，就不扯了。后面规范都扯[[GetOwnProperty]]，本来就不舒服，还绕来绕去的，看的头晕。。。
