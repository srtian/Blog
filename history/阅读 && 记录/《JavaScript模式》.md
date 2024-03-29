
## 第一章 概述

1. 在 JS 中定义变量的时候，其实也在和对象打交道。首先，变量自动变为一个被称为“活动对象”的内置属性（如果是全局变量的话，就变成全局对象的属性）。第二。这个变量实际上也是‘伪对象’，因为它自己的属性（属性特性），用以表示变量是否可以被修改，删除或在 foe-in 循环中枚举。
2. 对象有两大类：
- 本地对象：由 ES 标准规范定义的对象。
- 宿主对象：有宿主环境创建的对象（比如浏览器环境）

3. 本地对象也可以被归类为内置对象或自定义对象。宿主对象包括 window 和所有 DOM 对象
4. 组合优于继承。如果你手头有创建这个对象所需的资源，更推荐直接将这些资源组装成你所需的对象，而不推荐先作分类再创建链式父子继承的方式来创建对象。
5. 原型是一个普通的对象，所创建的每一个函数会自动带有 prototype 属性，这个属性指向一个空对象，这个空对象包含一个 constructor 属性，它指向你新建的函数而不是内置的 object(),除此之外它和通过对象直接量或 object()构造函数创建的对象没什么两样。


## 第二章 高质量的代码


### 前言

由于一般从项目而言，产品是不断进行迭代的，因此我们在大多数情况下都需要回去重读写好的代码。但往往由于各种原因，使得我们或者他人在阅读一个花了几个小时写的代码的时候，却往往需要花上好几天甚至好几周。因此编写可维护的代码对于我们而言是很重要的事情，而可维护的代码则意味着代码是：

- 可读的
- 一致的
- 可预测的
- 看起来像是同一个人写的
- 有文档的

这章将从几个方面来讨论如何编写可维护的代码。


### 减少全局变量

我们都知道 JavaScript 利用函数来管理作用域，而在函数作用域中定义的变量被我们称之为局部变量，通常而言我们需要使用函数将变量封装起来，以免将其暴露在全局作用域中。（现在一般使用 ES6 中的 let 和 const）


## 第三章 直接量和构造函数

1.对象直接量语法包括：

- 将对象主体包含在一对花括号内
- 对象内的属性或方法之间使用逗号分隔。最后一个名值对后也可以有逗号，但在IE下会报错，所以尽量不要在最后一个属性或方法后加逗号
- 属性名和值之间使用冒号分隔
- 如果将对象赋值给你个变量，不要忘记在右括号}之后补上分号<br />
2.这是两种创建对象的方式：

从上图可以清晰的看出直接量写法代码量比第二种少，且他可以强调对象是一个简单的可变的散列表，而不必一定派自某个类。另一个推荐使用直接量而不是Object构造函数创建实例函数的原因是，对象直接量不需要“作用域解析”，因为新创建的实例有可能包含一个本地的构造函数，打你调用Object()的时候，解析器需要顺着作用域链从当前作用域开始查找，直到找到全局Object()构造函数为止。

3.Object()构造函数可以接受阐述，通过参数的设置可以把实例对象的创建委托给另一个内置构造函数，并返回另外一个实例对象。

4.除了以上两种还可以通过自定义的构造函数来创建实例对象。当通过关键字new来调用构造函数时，函数体内将发生这些事情：

1. 创建一个空对象，将它的引用赋给this,继承函数的原型
2. 通过this将数据和方法添加至这个对象
3. 最后返回this指向的新对象（如果没有手动放回其他的对象）

5.用new调用的构造函数总是会返回一个对象，默认返回this所指向的对象。如果构造函数内没有给this赋给任何属性，则返回一个“空”对象（除了继承构造函数的原型之外，没有“自己的”属性）景观不会在构造函数内写return语句，也会隐式返回this。

6.如果调用构造函数时忘记new会发生什么呢？漏掉new不会产生语法错误，也不会有运行时错误，但可能会造成逻辑错误，导致执行结果不符合预期。这是因为如果不写new的话，函数内的this会指向全局对象。如下：

7.数组直接量写法：整个数组使用方括号括起来，数组元素之间使用逗号分隔。数组元素可以是任意类型，也包括数组和对象。


## 第四章函数
1.JS的函数具有两个主要特性、第一个特性是，函数是一等对象。第二个是函数提供作用域支持。<br />2.函数是对象，那么：

- 可以在程序执行时动态创建函数
- 可以将函数赋给变量，可以将函数的引用拷贝至另一个变量，可以扩充函数，除了某些特殊场景外均可以被删除
- 可以将函数作为参数传入另一个函数，也可以被当做返回值返回
- 函数可以包含自己的属性和方法
