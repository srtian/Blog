在早期编写JavaScript时，我们只需在 script 标签内写入JavaScript的代码就可以满足我们对页面交互的需要了。但随着时间的推移，时代的发展，原本的那种简单粗暴的编写方式所带来的诸如逻辑混乱，页面复杂，可维护性差，全局变量暴露等问题接踵而至，前辈们为了解决这些问题提出了很种的解决方案，其中之一就是JavaScript模块化编程。总的来说，它有以下四种优点：

1. 解决项目中的全局变量污染的问题。
2. 开发效率高，有利于多人协同开发。
3. 职责单一，方便代码复用和维护 。
4. 解决文件依赖问题，无需关注引用文件的顺序。


### 一、先行者CommonJs

2009年Node.js横空出世，将JavaScript带到了服务器端领域。而对于服务器端来说，没有模块化那可是不行的。因此CommonJs社区的大牛们开始发力了，制定了一个与社区同名的关于模块化的规范——CommonJs。它的规范主要如下：

1. 模块的标识应遵循的规则（书写规范）。
2. 定义全局函数require，通过传入模块标识来引入其他模块，执行的结果即为别的模块暴露出来的API。
3. 如果被require函数引入的模块中也包含依赖，那么依次加载这些依赖。
4. 如果引入模块失败，那么require函数应该报一个异常。
5. 模块通过变量exports来向外暴露API，exports只能是一个对象，暴露的API须作为此对象的属性。

根据CommonJS规范的规定，每个文件就是一个模块，有自己的作用域，也就是在一个文件里面定义的变量、函数、类，都是私有的，对其他文件是不可见的。通俗来讲，就是说在模块内定义的变量和函数是无法被其他的模块所读取的，除非定义为全局对象的属性。

```
// addA.js
const a = 1;
const addA = function(value) {
  return value + a;
}
```

上面代码中，变量a和函数addA，是当前文件addA.js私有的，其他文件不可见。如果想在多个文件中分享变量a，必须定义为global对象的属性：

```
global.a = 1;
```

这样我们就能在其他的文件中访问变量a了，但这种写法不可取，输出模块对象最好的方式是module.exports：

```
// addA.js
var a = 1;
var addA = function(value) {
  return value + x;
}
module.exports.addA = addA;
```

上面代码通过module.exports对象输出了一个函数，该函数就是模块外部与内部通信的桥梁。加载模块需要使用require方法，该方法读取一个文件并执行，最后返回文件内部的module.exports对象。

```
var example = require('./addA.js');
console.log(example.addA(1));  //2
```

CommonJs看起来是一个很不错的选择，拥有模块化所需要的严格的入口和出口，看起来一切都很美好，但它的一个特性却决定了它只能在服务器端大规模使用，而在浏览器端发挥不了太大的作用，那就是同步！这在服务器端不是什么问题，但放在浏览器端就出现问题了，因为文件都放在服务器上，如果网速不够快的话，前面的文件如果没有加载完成，浏览器就会失去响应！因此为了在浏览器上也实现模块化得来个异步的模块化才行！根据这个需求，我们的下一位主角——AMD就产生了！


### 二、AMD 异步模块定义

AMD的全名叫做：Asynchronous Module Definition即异步模块定义。它采用了异步的方式来加载模块，然后在回调函数中执行主逻辑，因此模块的加载不影响它后面的模块的运行。它的规范如下：

```
define(id?, dependencies?, factory);
```

1. 用全局函数define来定义模块;
2. id为模块标识，遵从CommonJS Module Identifiers规范
3. dependencies为依赖的模块数组，在factory中需传入形参与之一一对应
4. 如果dependencies的值中有"require"、"exports"或"module"，则与commonjs中的实现保持一致
5. 如果dependencies省略不写，则默认为["require", "exports", "module"]，factory中也会默认传入require,exports,module
6. 如果factory为函数，模块对外暴漏API的方法有三种：return任意类型的数据、exports.xxx=xxx、module.exports=xxx
7. 如果factory为对象，则该对象即为模块的返回值

具体分析AMD我们通过require.js来进行。require.js是一个非常小巧的JavaScript模块载入框架，是AMD规范最好的实现者之一，require.js的出现主要是来解决两个问题：

1. 实现JavaScript文件的异步加载，避免网页失去响应。
2. 管理模块的依赖性，管理模块的相互独立性，也就是我们常说的低耦合，这有利于代码的编写与维护。

使用require.js我们首先要加载它，为了避免浏览器未响应，我们在后面可以加上async,告诉浏览器这个文件需要异步加载（IE不支持该属性，所以需要把defer也加上）：

```
<script src="js/require.js" defer async="true" ></script>
```

定义模块时，在require.js中我们可以使用define，但define对于需要定义的模块是否是独立的模块的写法是不同;所谓的独立模块就是指不依赖于其他模块的模块，而非独立模块就是指不依赖于其他模块的模块。

define在定义独立模块时有两种写法，一种是直接定义对象；另一种是定义一个函数，在函数内的返回值就是输出的模块了：

```
define({
    method1: function() {},
    method2: function() {},
});
//等价于
define(function () {
	return {
	    method1: function() {},
		method2: function() {},
    }
});
```

如果define定义非独立模块，那么它的语法就规定一定是这样的：

```
define(['module1', 'module2'], function(m1, m2) {

    return {
        method: function() {
            m1.methodA();
			m2.methodB();
        }
    }

});
```

define在这个时候接受两个参数，第一个参数是module是一个数组，它的成员是我们当前定义的模块所依赖的模块，只有顺利加载了这些模块，我们新定义的模块才能成功运行。第二个参数是一个函数，当前面数组内的成员全部加载完之后它才运行，它的参数m与前面的module是一一对应的。这个函数必须返回一个对象，以供其他模块调用，需要注意的是，回调函数必须返回一个对象，这个对象就是你定义的模块。

在加载模块方面，AMD和CommonJs都是使用require。require.js也同样如此，它要求两个参数：module，callback：

```
require([module], callback);
```

第一个参数[module]，是一个数组，里面的成员就是需要加载的模块；第二个参数callback，则是加载成功之后的回调函数。<br />require方法本身也是一个对象，它带有一个config方法，用来配置require.js运行参数。config方法接受一个对象作为参数。

```
//别名配置
requirejs.config({
    paths: {
        jquery: [   //如果第一个路径不能完成加载，就调到第二个路径继续进行加载
            '//cdnjs.cloudflare.com/ajax/libs/jquery/2.0.0/jquery.min.js',
            'lib/jquery'   //本地文件中不需要写.js
        ]
    }
});

//引入模块，用变量$表示jquery模块
requirejs(['jquery'], function ($) {
    $('body').css('background-color','black');
});
```

虽然require.js实现了异步的模块化，但它仍然有一些不足的地方，在使用require.js的时候，我们必须要提前加载所有的依赖，然后才可以使用，而不是需要使用时再加载，使得初次加载其他模块的速度较慢，提高了开发成本。


### 三、CMD 通用模块定义

CMD的全称是Common Module Definition，即通用模块定义。它是由蚂蚁金服的前端大佬——玉伯提出来的，实现的JavaScript库为sea.js。它和AMD的require.js很像，但加载方式不同，它是按需就近加载的，而不是在模块的开始全部加载完成。它有以下两大核心特点：

1. 简单友好的模块定义规范：Sea.js 遵循 CMD 规范，可以像 Node.js 一般书写模块代码。
2. 自然直观的代码组织方式：依赖的自动加载、配置的简洁清晰，可以让我们更多地享受编码的乐趣。

在CMD规范中，一个文件就是一个模块，代码书写的格式是这样的：

```
define(factory);
```

当factory为函数时，表示模块的构造方法，执行该方法，可以得到该模块对外提供的factory接口，factory 方法在执行时，默认会传入三个参数：require、exports 和 module：

```
// 所有模块都通过 define 来定义
define(function(require, exports, module) {

  // 通过 require 引入依赖
  var $ = require('jquery');
  var Spinning = require('./spinning');

  // 通过 exports 对外提供接口
  exports.doSomething = ...

  // 或者通过 module.exports 提供整个接口
  module.exports = ...

});
```

它与AMD的具体区别其实我们也可以通过代码来表现出来，AMD需要在模块开始前就将依赖的模块加载出来，即依赖前置；而CMD则对模块按需加载，即依赖就近，只有在需要依赖该模块的时候再require就行了：

```
// AMD规范
define(['./a', './b'], function(a, b) {  // 依赖必须一开始就写好  
   a.doSomething()    
   // 此处略去 100 行    
   b.doSomething()    
   ...
});
// CMD规范
define(function(require, exports, module) {
   var a = require('./a')   
   a.doSomething()   
   // 此处略去 100 行   
   var b = require('./b') 
   // 依赖可以就近书写   
   b.doSomething()
   // ... 
});
```

需要注意的是Sea.js的执行模块顺序也是严格按照模块在代码中出现(require)的顺序。

从运行速度的角度来讲，AMD虽然在第一次使用时较慢，但在后面再访问时速度会很快；而CMD第一次加载会相对快点，但后面的加载都是重新加载新的模块，所以速度会慢点。总的来说,<br />require.js的做法是并行加载所有依赖的模块, 等完成解析后, 再开始执行其他代码, 因此执行结果只会"停顿"1次, 而Sea.js在完成整个过程时则是每次需要相应模块都需要进行加载，这期间会停顿是多次的，因此require.js从整体而言相对会比Sea.js要快一些。

### 四、ES6模块特性

在ES6中将模块认为是自动运行在严格模式下并且没有办法退出运行的JavaScript代码。在一个模块中定义的变量不会自动被添加到全局共享的作用域之中，这个变量只能作用在这个作用域中。此外模块还必须导出一些外部文件可以访问的元素，以供其他模块或代码使用。

除了这个基本特性，ES6模块还有两大特性也十分重要，需要额外注意：

- 首先是在模块的顶部this值是undefined，这是由于在ES6中的模块的代码是在严格模式下执行的。（如果对this不是很熟悉的可以去看我的这篇文章[：深入浅出this关键字](https://www.jianshu.com/p/8d6cc7ad9c58/)）
- 其次，模块不支持HTML风格的代码注释，这是早期浏览器所遗留下的JavaScript特性，在ES6的语法里不予支持。

#### 4.1、基本用法-模块加载

首先我们来看浏览器是如何加载模块的。其实在ES6规范出来之前，web浏览器就规定了三种方式来引入JavaScript文件：

- 在没有src属性的 script 元素中直接内嵌JavaScript代码
- 在 script 元素中通过src属性指定一个地址来加载JavaScript代码文件
- 通过Web Worker或Service Worker的方法加载并执行JavaScript代码

而在浏览器中，默认的行为就是将JavaScript作为脚本来进行加载，而非模块。所以我们要告诉浏览器我们加载的是模块，方法就是在 script 元素中，将type属性指定为"module"。具体看下面的示例：

```
// 第一种方式
<script type=""module>
   import { add } from "./example";
   let num = add(1, 1);
</script>
//  第二种方式
<script type="module" src="example.js">
// 第三种方式，以脚本的方式加载example.js
let worker = new Worker("example.js");
```

当HTML解析器遇到 script 元素的type="module"的时候，模块文件就开始下载，直到文件被完全解析完成才会去执行模块内的代码。模块文件是按照他们出现在HTML文件中顺序执行的，也就是说无论用何种方式引入模块，第一个 script type="module" 总是在第二个 script type="module" 之前执行。


#### 4.2、基本用法-导出

在ES6中我们可以使用export关键字将一部分代码暴露给其他模块，以供其他模块或代码使用。先让我们来看看export关键字在MDN的定义吧：

> export语句用于在创建JavaScript模块时，从模块中导出函数、对象或原始值，以便其他程序可以通过 import 语句使用它们。（
> 此特性目前仅在 Safari 和 Chrome 原生实现。它在许多转换器中实现，如Traceur Compiler，Babel或Rollup。）
> 通过MDN的定义我们可以知道：export关键字可以将其放在任何函数、对象或原始值前面，从而将它们从模块中导出。示例如下：


```
//   ./example.js
// 导出变量
export var a = 1;
// 导出函数
export function addA(value) {
   return value + a;
}
//导出类
export class add1 {
   constructor(value) {
       this.value = value + a;
   }
}
//这个函数就是这个模块所私有的，在外部不能访问它
function say1() {
   console.log('我是不是很帅');
}
//这又是个函数
function say2() {
   console.log('没错我就是很帅');
}
//在后面对函数进行导出,它就不是私有的了
export say2;
```

需要注意的是：使用export导出的函数和类都需要一个名称，除非使用default关键字，否则就不能用这个方法导出匿名函数或类。所以当我们需要导出匿名的函数或者类时，我们可以这么做：

```
//   ./example.js
//导出匿名函数
export default function(a, b) {
   return a + b；
}
//或者导出匿名的类
export default class {
consturctor(value) {
   this.value = value + 1;
   }
}
```

具体关于default关键字的用法我会在后面做具体介绍，现在只需记住：当我们需要导出匿名的函数或者类时要使用export default语法。


#### 4.3、基本语法-导入

在ES6中，从模块中导入的功能可以通过import关键字。import语句由两部分组成：要导入元素的标识符和元素应当从哪个模块导入。

```
//  ./say.js
import { say2 } from "./example.js";
console.log(say2()); // '没错我就是很帅'
```

import 后面的大括号中的say2表示从规定模块导入的元素的名称。关键字from后面的字符串则表示要导入的模块的路径，这通常是包含模块的.js文件的相对或绝对路径名，需要注意的是只允许使用单引号和双引号的字符串来包裹路径，浏览器使用的路径格式与传给 script 元素的相同，所以必须把文件的扩展名也加上。

> （注：由于Node.js遵循基于文件系统前缀以区分本地文件个包的惯例，即example是一个包，而./exampple.js是一个本地文件。为了更好的兼容多个浏览器Node.js环境，我们一定要在路径前包含./或../来表示要导入的文件。）
> 除此之外，我们还可以导入多个元素或者直接导入整个模块：


```
// 导入多个元素
improt { a, addA, say2 } from "./example.js";
console.log(a); // 1
sonsole.log(addA(1); // 2
// 导入整个模块
import * as example from "./example.js"
console.log(example.a); // 1
sonsole.log(example.addA(1); // 2
console.log(example.say2()); // '没错我就是很帅'
```

上面的导入整个模块就是把example.js中导出的所有元素全部加载到一个叫做example的对象中，而所导出的元素就会作为example的属性被访问。因为example对象是作为example.js中所导出成员的命名空间对象而被创建的，所以这种导入方式被称为命名空间导入（name space import)。<br />还有一点要注意的是，不管import语句把一个模块写了多少次，该模块只执行一次。意思就是，在首次执行导入模块后，实例化的模块就会被保存在内存中，只要使用import语句引用它就可以重复使用它：

```
// 首次导入需要加载模块example.js
import { a } from "./example.js"
// 下面的两个import将无需加载example.js了
import { addA } from "./example.js"
import { say2 } from "./example.js"
```

当从模块中导入一个元素时，它与const是一样无法定义另一个同名变量和导入一个同名元素，也无法在import语句前使用元素或者改变导出的元素的值：

```
//接上面的代码
say2 = 1 ;  //会抛出一个错误
```

这是由于ES6的import语句为导入的元素创建的是只读绑定的标识符，而不是原始绑定。因此元素只有在被导出的模块中才可以被修改，即使是将该模块的全部导入也无法修改其中的元素。

```
//   ./example.js
// 这是一个函数
export function setA(newA) {
   a = newA;
}
//  ./say.js
import { a, setA } from "./example";
console.log(a);  // 1
a = 2;   //抛出错误
// 所以我们得这么做
setA(2);
console.log(a);  // 2
```

调用setA(2)时会返回到example.js中去执行，将a设置为2。由于say.js导入的只是a的只读绑定的标识符而已，因此会自动进行更改。


#### 4.4、其他基本语法


##### 1.语法限制

export和import在语法上还有一个重要的限制，那就是他们必须在条件语句和函数之外使用，例如：

```
if (ture) {
   export var a = 1;      //语法错误
}
function imp() {
   import a from "./example.js"; //语法错误
}
```

由于模块语法存在的其中一个原因是让JavaScript引擎可以静态地确定哪些代码是可以导出的，因此export和import语句被设计成静态的，不能进行任何形式的动态导出或导入。


##### 2.重命名解决

有时在开发中，我们在导入一些元素后不想使用它们的原始名称了，我们就可以在导出过程或者导入过程中去改变导出元素的名称：

```
// 导出过程
function add(a, b) {
   return a + b;
}
export { add as add1 };  //在导入过程中必须使用add1作为名称
// 导入过程
import {add as add1 } from "./example"
console.log(add1(1,1));  // 2
console.log(typeof add); //undefined
```


##### 3.模块的默认值

在CommonJS等其他的模块化规范中，从模块中导出或导入默认值是一个常见的用法，因此在ES6中也延用了这种用法并进行了优化。在ES6中我们可以使用default关键字来指定默认值，并且一个模块只能默认一个导出值：

```
// ./example.js
// 第一种默认导出语法
export default function(a, b) {
   return a + b;
}
// 第二种默认导出语法
function add(a, b) {
   return a + b;
}
export default add;
// 第三种默认导出语法
function add(a, b) {
   return a + b;
}
export { add as default };
```

需要注意的是第三种语法，default关键字虽然不能作为元素的名称，但可以作为元素的属性名称，因此可以使用as语法将add函数的属性设置为default。<br />导入默认值的语法则是这样的：

```
//  第一种语法
import add from "./example";
//  第二种语法
import { default as add } from "./example";
```

看到这里有些朋友可能会发现，我们的第一种语法中import关键字后面并没有加大括号，认为这是错误的。其实这是导入默认值的独特语法，在这的本地名称add用于表示模块导出的任何默认函数，这种语法是最纯净的，ES6标准创建团队的大佬们也希望这种语法能成为web主流的模块导入形式。<br />我们前面说的导入匿名函数也同样使用这种语法：

```
//   ./example.js
//导出匿名函数
export default function(a, b) {
   return a + b；
}
// ./say.js
import add from "./example";
console.log(add(1,1));  // 2
```

在这里本地名称add就是用于表示上面的匿名函数的。


##### 4.导出已导入的元素

我们同样可以在本模块内导出我们在本模块内导入的元素，有以下几种语法：

```javascirpt
//  第一种语法
import { add } from ./example.js;
export { add };
//  第二种语法
export { add } from ./example.js;
//换一个名称导出
export { add as add1 } from ./example.js; //以add这个名称导入，再以add1的名称导出
// 导出整个模块
export *  from ./example.js;
```

// 最后求一波暑期前端实习的坑位-_-||
