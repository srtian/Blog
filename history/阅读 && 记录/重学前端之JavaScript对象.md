
## 一、基于对象的JavaScript
JavaScript是一门基于对象的编程语言，其标准对基于对象的定义如下：
> 语言和宿主的基础设施由对象来提供，并且 JavaScript 程序即是一系列互相通讯的对象集合”

通过下图，也可以潜窥对象之于JavaScript的重要性：

![](https://cdn.nlark.com/yuque/0/2020/svg/296173/1585536244224-f652545d-6b3a-4ba1-bf41-a0bcc1c37078.svg)
### 1、面向对象

#### 1.1、什么是面向对象
对象其实并不是计算机领域凭空造出来的概念，而是对于人类思维模式的一种抽象，在《面向对象分析与设计》中从人类的认知角度出发，对象应该是以下事物的一种：

- 一个可以触摸或者可以看见的东西；
- 人的智力可以理解的东西；
- 可以指导思考或行动（进行想象或施加动作）的东西

有了对对象的基本定义，语言的设计者们就可以通过相应的语言特性来对对象进行描述来，其最主要的实现方式有两种：

1. 基于类的对象描述方式（典型代表Java）
2. 基于原型的对象描述方式(典型代表JavaScript)


#### 1.2、JavaScript对象特征
《面向对象分析与设计》一书中对于对象特征的描述主要有以下三点：

- 对象具有唯一标识性：即使完全相同的两个对象，也并非同一个对象。
- 对象有状态：对象具有状态，同一对象可能处于不同状态之下。
- 对象具有行为：即对象的状态，可能因为它的行为产生变迁。

其中第一点，对象具有唯一标识性。一般来说，各种语言的对象唯一标识性都是用内存地址来体现的， 对象具有唯一标识的内存地址，所以具有唯一的标识，JavaScript也不例外：

```javascript
var o1 = { a: 1 };
var o2 = { a: 1 };
console.log(o1 == o2); // false
```

让对于对象的第二和第三哥特征，在不同语言中也会用不同的术语去描述他们，比如C++ 中的“成员变量”和“成员函数”，Java 中的属性”和“方法”。但JavaScript则和他们不同，它将状态和行为统一抽象为“属性”，这是由于在JavaScript中，函数其实也是作为一种特殊的对象存在，因此状态和行为都可以统一用属性来抽象。比如下面的代码，在javaScript中，d以及f都是属性，对于其对象o来说，并无太大区别。

```javascript
var o = {
  d: 1,
  f() {
    console.log(this.d)
  }
}
```

其次在JavaScript中，对象具有高度的动态性，JavaScript赋予了使用者可以在运行时为对象动态的添加状态和行为的能力。而为了提高抽象能力，JavaScript属性又被设计为两大类，并用一组特征来描述属性：

- 数据属性
- 访问器属性

**数据属性**

- value：就是属性的值。
- writable：决定属性能否被赋值。
- enumerable：决定 for in 能否枚举该属性。
- configurable：决定该属性能否被删除或者改变特征值。

**访问器属性**

- getter：函数或 undefined，在取属性值时被调用。
- setter：函数或 undefined，在设置属性值时被调用。
- enumerable：决定 for in 能否枚举该属性。
- configurable：决定该属性能否被删除或者改变特征值。

访问器属性使得属性在读和写时执行代码，它允许使用者在写和读属性时，得到完全不同的值。VUE2.0就是利用来对象的这一特性来实现双向绑定的。当我们想要改变属性的特征，或者定义访问器属性，我们可以使用 Object.defineProperty：

```javascript
var o = { a: 1 };
Object.defineProperty(o, "b", {value: 2, writable: false, enumerable: false, configurable: true});
//a 和 b 都是数据属性，但特征值变化了
Object.getOwnPropertyDescriptor(o,"a"); // {value: 1, writable: true, enumerable: true, configurable: true}
Object.getOwnPropertyDescriptor(o,"b"); // {value: 2, writable: false, enumerable: false, configurable: true}
o.b = 3;
console.log(o.b); // 2
```

因此，我们可以将对象的运行时理解为一个“属性的集合”，属性以字符串或Symbol为key,以树枝属性特征值或访问器属性特征值为value。

因此由上我们不难看出，虽然JavaScript实现对象的方式与其他的传统的面向对象的编程语言Java等不一样，但其实本质上都满足来对象的基本特征，因此将JavaScript归类为面向对象的编程语言其实并不为过。

### 2、基于原型实现的对象
上面以及说明JavaScript是基于原型实现了自身的对象系统，那它和基于类所实现的对象系统有什么区别呢？

首先“基于类”，就是提倡使用一个关注和类之间关系开发模型，在这类实现中，总是先有类，再从类去实例化一个对象。而类与类之间又会形成继承，组合等关系。类又往往与语言的类型系统整合，形成一定编译时的能力。

而基于“原型”则更提倡程序员去关注一系列对象实例的行为，然后再去关心如何将这些对象，划分到最近使用方式相似的原型对象，而不是将他们分成类。基于原型的面向对象的系统通过复制的方式来创建新的对象，从实现上来讲，原型系统的“复制操作”有两种实现思路：

- 并不真的去复制一个原型对象，而是使得新对象持有一个原型的引用；<br />
- 切实地复制对象，从此两个对象再无关联。<br />

很明显JavaScript是基于第一种方式实现复制操作的。


#### 2.1、JavaScript原型
抛开JavaScript模拟Java的那些语法，原型系统可以用两句话来总结：

- 如果所有对象都有私有字段 [[prototype]]，就是对象的原型；
- 读一个属性，如果对象本身没有，则会继续访问对象的原型，直到原型为空或者找到为止。

这个模型并无太大的区别，到了ES6提供了三个内置函数来直接访问和操作原型：

- Object.create 根据指定的原型创建新对象，原型可以是 null；
- Object.getPrototypeOf 获得一个对象的原型；
- Object.setPrototypeOf 设置一个对象的原型。

具体使用如下：
```javascript
var cat = {
    say(){
        console.log("meow~");
    },
    jump(){
        console.log("jump");
    }
}
 
var tiger = Object.create(cat,  {
    say:{
        writable:true,
        configurable:true,
        enumerable:true,
        value:function(){
            console.log("roar!");
        }
    }
})
 
 
var anotherCat = Object.create(cat);
 
anotherCat.say();
 
var anotherTiger = Object.create(tiger);
 
anotherTiger.say();
```

#### 2.2、早期的类和原型
在早期版本（ES3以及更早的版本）中，类的定义是一个私有属性[[class]]，语言标准为内置类型Number等都制定了[[class]]属性，而我们唯一能访问它的方式就是Object.prototype.toString，我们通常也是使用这个方式来判断数据类型。因此，在早期版本中，类是一个相当容的概念，仅仅是运行时的一个字符串属性。

在 ES5 开始，[[class]] 私有属性被 Symbol.toStringTag 代替，Object.prototype.toString 的意义从命名上不再跟 class 相关。我们甚至可以自定义 Object.prototype.toString 的行为，以下代码展示了使用 Symbol.toStringTag 来自定义 Object.prototype.toString 的行为：

```javascript
var o = { [Symbol.toStringTag]: "MyObject" }
console.log(o + "");
```
 对于new，我们不能说“new 运算是针对构造器对象，而不是类”，我们也要将其理解成JavaScript的面向对象的一部分。

new运算接受一个构造器和一组调用参数，实际上做了几件事：

- 以构造器的prototype属性（注意与私有[[prototype]]的区分）为原型，创建新对象；
- 将this和调用参数传给构造器，执行；
- 如果构造器返回的是对象，则返回，否则返回第一步创建的对象。

new的出现，主要是试图让函数对象在语法上和类相似，它提供了两种方式来为对象添加属性：

1. 构造器中添加属性
2. 在构造器的prototype属性上添加属性

```javascript
function c1(){
    this.p1 = 1;
    this.p2 = function(){
        console.log(this.p1);
    }
} 
var o1 = new c1;
o1.p2();
 
 
 
function c2(){
}
c2.prototype.p1 = 1;
c2.prototype.p2 = function(){
    console.log(this.p1);
}
 
var o2 = new c2;
o2.p2();
```
在没有 Object.create、Object.setPrototypeOf 的早期版本中，new 运算是唯一一个可以指定 [[prototype]] 的方法。所以就有人使用这个方法来代替Object.create，比如这种Object.create的polyfill:
```javascript
Object.create = function(prototype){
  var cls = function(){}
  cls.prototype = prototype
  return new cls
}
```

这段代码就是创建了一个空函数作为类，并把传入的原型挂在他的prototype上，最后创建一个它的实例，这就产生例一个以传入的第一个参数作为原型的对象。


#### 2.3、ES6的类
ES6中引入了class关键字，并在标准中删除了所有[[class]]相关的私有属性描述，类的概念正式从属性升级成为了语言的基础设施。

```javascript
class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
  // Getter
  get area() {
    return this.calcArea();
  }
  // Method
  calcArea() {
    return this.height * this.width;
  }
}
```

### 3、JavaScript对象分类

JavaScript的对象大致可以分为以下几类：

- 宿主对象（host object）: 由JavaScript宿主环境提供对象，他们的行为完全由宿主环境提供。
- 内置对象（built in Object): 由JavaScript语言提供的对象
   - 固有对象（Intrinsic Objects）：由标准规定，随着JavaScript运行时创建而自动创建的对象实例。
   - 原生对象（Native Obejct）: 可以由对象通过Array，RegExp等内置构造器或特殊语法创建的对象
   - 普通对象（Ordinary Object）:由{}语法，Obejct构造器或者Class关键字定义类创建的对象，他能够被原型继承。


#### 3.1、宿主对象

JavaScript宿主对象会随着其运行环境的不同而不同，但前端最熟悉的当属浏览器环境中的宿主了。

在浏览器中，我们所熟悉的全局对象是window，而window上又有很多属性，如document。实际上，这个全局对象window上的属性，一部分来自javaScript语言本身，一部分来自浏览器环境。JavaScript标准规定了全局对象属性，w3c的各种标准中规定了window对象的其他属性。

宿主对象也分为固有的以及用户可创建两种，比如document.createElement就可以创建一些dom对象。宿主对象也会提供一些构造器，比如我们可以使用 new Image 来创建 img 元素。


#### 3.2、内置对象·固有对象

固有对象是由标准规定的，随着JavaScript运行时创建而自动创建的对象实例。

固有对象在任何JS代码执行前就已经被创建出来了，他们通常扮演着基础库的角色。我们常说的类就是固有对象的一种，ECMA标准为我们提供类一份固有对象表（150+）：

> [https://www.ecma-international.org/ecma-262/9.0/index.html#sec-well-known-intrinsic-objects](https://www.ecma-international.org/ecma-262/9.0/index.html#sec-well-known-intrinsic-objects)



#### 3.3、原生对象

我们吧JavaScript中，能够通过语言本身的构造器创建的对象称为原生对象。在javaScript中，提供了30多个构造器：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/296173/1585204239995-b28b5bf7-52f6-46a4-be1a-bfde1779c723.png#align=left&display=inline&height=188&name=image.png&originHeight=375&originWidth=988&size=99855&status=done&style=none&width=494)<br />这些构造器创建的对象多数使用了私有字段：

- Error: [[ErrorData]]
- Boolean: [[BooleanData]]
- Number: [[NumberData]]
- Date: [[DateValue]]
- RegExp: [[RegExpMatcher]]
- Symbol: [[SymbolData]]
- Map: [[MapData]]

这些字段使得原型继承方法无法正常工作，所以，我们可以认为，所有这些原生对象都是为了特定能力或者性能，而设计出来的“特权对象”


#### 3.4、用对象来模拟函数与构造器：函数对象与构造器对象

在 JavaScript 中，还有一个看待对象的不同视角，这就是用对象来模拟函数和构造器。JavaScript 为这一类对象预留了私有字段机制，并规定了抽象的函数对象与构造器对象的概念。

函数对象的定义是：具有 [[call]] 私有字段的对象，构造器对象的定义是：具有私有字段 [[construct]] 的对象。

JavaScript 用对象模拟函数的设计代替了一般编程语言的函数，它们可以像其他语言的函数一样被调用，传参。任何宿主只要提供了[[call]]私有字段的对象，就可以被 JavaScript 函数调用语法支持。

> [[call]] 私有字段必须是一个引擎中定义的函数，需要接受 this 值和调用参数，并且会产生域的切换，这些内容，我将会在属性访问和执行过程两个章节详细讲述。


我们可以说任何对象只要实现了[[call]]，那么他就是一个函数对象，而如果实现了[[construct]]，他就是一个构造器对象，可以作为构造器对象使用

> 对于这两个方法，我们也可以对其进行设置，来定制对象的具体调用方式的限制


对于宿主和内置对象来讲，，他们实现[[call]]和[[construct]]不总是一致的。比如内置对象 Date 在作为构造器调用时产生新的对象，作为函数调用是则产生字符串。

```javascirpt
console.log(new Date); // 1
console.log(Date())
```

而浏览器宿主环境中，提供的 Image 构造七，则不允许作为函数调用。

需要注意的是，在 ES6 之后的箭头函数语法创建的函数仅仅只是函数，不能作为构造器使用。而对用使用 function 语法或者 Function 构造器创建的对象来讲，[[call]] 和 [[construct]] 的行为总是相似的。

我们可以大致认为，他们[[construct]]的执行过程如下：

- 以 Object.prototype 为原型创建一个新对象
- 以新对象为 this, 执行函数的[[call]]
- 如果[[call]]的返回值是对象，那么就返回这个对象，否则返回第一步创建的对象。

这样的规则造成了个有趣的现象，如果我们的构造器返回了一个新的对象，那么 new 创建的新对象就变成了一个构造函数之外完全无法访问的对象，这一定程度上可以实现“私有”。

```javascirpt
function cls(){
    this.a = 100;
    return {
        getValue:() => this.a
    }
}
var o = new cls;
o.getValue(); //100
//a 在外面永远无法访问到
```

一些特殊行为的对象：

- Array：Array 的 length 属性根据最大的下标自动发生变化。
- Object.prototype：作为所有正常对象的默认原型，不能再给它设置原型了。
- String：为了支持下标运算，String 的正整数属性访问会去字符串里查找。
- Arguments：arguments 的非负整数型下标属性跟对应的变量联动。
- 模块的 namespace 对象：特殊的地方非常多，跟一般对象完全不一样，尽量只用于 import 吧。
- 类型数组和数组缓冲区：跟内存块相关联，下标运算比较特殊。
- bind 后的 function：跟原来的函数相关联。
