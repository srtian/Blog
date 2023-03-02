---



## 一、概述

在JavaScript中，数组是数据类型中非常重要的一环。所传入的数据一般也都需要转化为数组，然后通过数组所提供的API来对数据进行处理，当然它可能本身传来的就是数组。此外，在函数式编程中，数组也提供了不少的API来帮助我们实现函数式编程，从而使代码变得更为可读，因此对数组的一些操作进行一个总结还是很有必要的。

由于在JavaScript中，数组的API还是挺多的，因此本文将主要讨论一些常用的API：

- Array.join
- Array.slice
- Array.sort
- Array.forEach
- Array.map
- Array.filter
- Array.reduce

其中Array.map、Array.filter、Array.reduce都是高阶函数。


## 二、数组操作


### 2.1 Array.join()

Array.join()是个非常简单的API，该方法将一个数组（或一个类数组对象）的所有元素连接成一个字符串并返回这个字符串。具体用法如下：

```javascript
let a = [1, 2, 3]
a.join('-') // "1-2-3"
```

其中join的工作其实很简单：即遍历数组，并将数组中的每一个元素都与传入的参数相加。模拟起来大致的运行流程就是这样：

```javascript
Array.prototype.join = function(char){
  let result = this[0] || ''
  let len = this.length
  for(let i = 1; i < len; i++){
      result += char + this[i]
  }
  return result
}
```

此外，join()也能连接类数组：

```javascript
function f(a, b, c) {
  var s = Array.prototype.join.call(arguments);
  console.log(s);
}

f(1, 'a', true)  // '1,a,true'
```


### 2.2 Array.slice

Array.slice(beginIndex, endIndex) 方法返回一个新的数组对象，这一对象是一个由 begin 和 end（不包括end）决定的原数组的浅拷贝,且原始数组不会被改变：

```javascript
let fruits = ['Banana', 'Orange', 'Lemon', 'Apple', 'Mango']
let citrus = fruits.slice(1, 3) // ['Orange','Lemon']
```

它的用法也很简单，因此在ES6出现之前，很多人都用它来讲类数组装化为数组：

```javascript
array = Array.prototye.slice.call(arrayLike)
// or
array = [].slice.call(arrayLike)
```

所以仔细想想Array.slice()的套路大概就是这样：

```javascript
Array.prototype.slice = function(begin, end){
    begin = begin || 0
    end = end || this.length
    let result = []
    for(let i = begin; i < end; i++){
        result.push(this[i])
    }
    return result
}
```

当然了，使用此方法来将类数组转化为数组非常的不优雅，因此ES6又提供了Array.from来讲类数组转化为数组：

```javascript
let a = array.from(arrayLike)
```


### 2.3 Array.sort

Array.sort([callback])也是数组操作中常用的一个API，其内置的排序算法主要是快速排序和插入排序的混合，当数组长度低于11个的时候就是用插入排序，除此之外就使用快排。具体代码可以直接去看源码：

> github.com/v8/v8/blob/master/src/js/array.js#L718


```javascript
function InnerArraySort(array, length, comparefn) {
  // In-place QuickSort algorithm.
  // For short (length <= 10) arrays, insertion sort is used for efficiency.
  if (!IS_CALLABLE(comparefn)) {
    comparefn = function (x, y) {
      if (x === y) return 0;
      if (%_IsSmi(x) && %_IsSmi(y)) {
        return %SmiLexicographicCompare(x, y);
      }
      x = TO_STRING(x);
      y = TO_STRING(y);
      if (x == y) return 0;
      else return x < y ? -1 : 1;
    };
}
```

注释也说的很清楚，默认使用快速排序，但是当数组长度小于11的数组，使用的是插入排序。这也符合我们的正常观点，因为插入排序在数组长度小于一定值的时候是会比快速排序速度更快，而快速排序在大规模数据的时候则更有优势。插入排序是稳定的，快速排序是不稳定的。当然这在不同的浏览器引擎中的表现是不同的，比如在火狐浏览器中使用的则是归并排序。

那么我们一般如何使用它呢？首先它的语法是这样的：Array.sort([callback])。也就是说我们可用通过定制callback也就是回调函数来定义如何对数组里的元素进行排序。但获取有些小老弟会发现sort有这么一个很有意思的特性，这也是我早期学JavaScript时，碰到的一个很有意思的问题：

```javascript
let numbers1 = [1, 3, 5, 2, 4]
numbers1.sort()
console.log(numbers1) // [1, 2, 3, 4, 5]
// 还有这样的
let numbers2 = [1, 15, 20, 4, 3]
numbers2.sort()
console.log(numbers2) // [1, 15, 20, 3, 4]
```

我们可以看到，同样是调用sort()方法，但上下两个数组所得到的结果完全不同。其实其根本原因在于sort方法执行的时候，数组的每个元素会先执行一次 toString() 方法，然后在根据字符串的 Unicode 编码进行排序。因此我们在使用sort的时候就需要定义相应的callback，来达到排序的目的，比如最常见的升序排序：

```javascript
let a = [1, 23, 5, 12, 4]
a.sort((a, b) => a - b)
consloe.log(a) // [1, 4, 5, 12, 23]
```

其计算规则很简单：

- 如果 a - b 小于 0 ，那么 a 在 b 的前面，也就是会按照升序排列
- 如果 a - b 等于 0 ，那么 a 和 b 的位置保持不变（相对的，视具体算法而变）
- 如果 a - b 大于 0 ，那么 b 在 a 的前面，也就是会按照降序排列

当然我们也不局限如此，我们经常碰到的数据可能不局限于单纯的数字数组，比如我在一次面试时，面试官就给了我这么一串数据：

```javascript
[{order: 3},{order: 4},{order: 2},{order: 100}]
```

让我将其根据order的值从大到小进行排序。既然而然我们就能使用sort来对其进行排序了：

```javascript
let order = [{order: 3},{order: 4},{order: 2},{order: 100}]
var result = order.sort((a, b) => b.order - a.order)
```

这样就能实现我们预期的效果了，是不是很强大、很简单~


### 2.4 Array.forEach

forEach这个API在我们日常开发中也很有用（当然其实我个人认为在map出来后可以使用map来代替forEach,这个在后面说为啥），它可以对数组的每一个元素都进行操作，比如这样：

```javascript
let a = [1, 2, 3]
a.forEach((item) => console.log(item + 1))
// 2 3 4
```

它的大致实现思路就像这样：

```javascript
Array.prototype.forEach = function(fn) {
    for (let i = 0, len = this.length; i < len ; i++) {
         fn.apply(this, [this[i], i, this])
    }
}
```

因此总的来说，forEach与我们常用的for循环在功能上存在重叠，也在一定程度上可以替代for循环的一些功能，但需要注意的是，它与for循环还是存在以下一些差异：

- forEach 没法 break
- forEach 用到了函数，因此每次迭代都会产生一个新的函数作用域；而 for 循环只有一个作用域


### 2.5 Array.map

map()方法在功能上和forEach相差不大，但由于它本身存在返回值，且调用起来也更为方便，代码可读性也更高，因此推荐使用map：

```javascript
let a = [1,2,3,4,5]
let b = a.map((item) => item + 1)
console.log(b)
```

同时，在函数式编程中，使用map保证存在一个返回值也是达到链式编程的一个重要的基础。map大概的代码逻辑应该是这样的：

```javascript
Array.prototype.map = function (fn) {
    let result = [];
    for (let i = 0, len = this.length; i < len; i++) {
         result[i] = fn.apply(this, [this[i], i, this]);
    }
    return result
}
```


### 2.6 Array.filter

filter() 方法创建一个包含所有通过测试的元素的新数组。我们通常使用filter()来对数组进行筛选，挑选出我们需要的元素，同时也可以使用代码更具有可读性：

```javascript
const prices = [25, 30, 15, 55, 40, 10]
prices.filter((item) => item >= 30)
// [30, 55, 40]
```

同样的，作为函数式编程重要的一员，filter同样也不会对原有的数组产生影响，而是产生一个新的数组，从而达到链式调用的目的，因此我们就可以猜想它的实现逻辑大致就是下面这样的：

```javascript
Arra.prototype.filter = function(fn){
    let result = []
    let temp
    for (let i=0; i<this.length; i++) {
        if(i in this) {
            if (temp = fn.call(undefined, this[i], i, this)) {
                result.push(this[i])
            }
        }
    }
    return result
}
```


### 2.7 Array.reduce

Array.reduce(callback,[initialValue])方法对累计器和数组中的每个元素（从左到右）应用一个函数，将其简化为单个值。它接受四个参数:

- Accumulator (acc) (累计器)
- Current Value (cur) (当前值)
- Current Index (idx) (当前索引)
- Source Array (src) (源数组)

执行起来就是这样的：

```javascript
[0, 1, 2, 3, 4].reduce(function(accumulator, currentValue, currentIndex, array){
  return accumulator + currentValue;
}) // 10
```

reduce的功能非常强大，它同样也可以模拟map以及filter，比如模拟map:

```javascript
let array = [1,2,3,4,5]
let array1 = array.map((v) => v+1)
//用reduce实现为：
let array2 = array.reduce((result, v)=> {
     result.push(v + 1)
     return result
 },[ ])
```

同样我们也能来模拟filter:

```javascript
let array1 = array.filter( (v) => v % 2 === 0 )
//用reduce实现为：
let array2 = array.reduce((result, v) => {
     if(v % 2 === 0){
        result.push(v)
        }
     return result
 }, [])
```

是不是很有意思~ 当然平常开发的时候别这么做，因为使用指定的API来完成指定的任务会让代码变得更可读，更以维护~


## 三、关于高阶函数的性能

关于函数式编程最近在社区发生了很多的讨论。以我个人的看法，函数式编程在开发一些普通的应用或当应用变大时，其声明式的、纯函数式的，都为项目代码的可维护性以及可读性提供了很好的支持，这也是react最近出来的hooks以及redux还有Rx.js如此火爆的原因。但同时，我们也应该更具具体情况来考虑使用怎样的编程方式，比如我们在解决一些CV或较为底层的开发的时候，函数式编程可能就不能满足我们的开发需求，毕竟函数式编程在性能方面还是有所缺失的，比如在上面所说的那些高阶函数与原生for循环之间在性能上区别:

> [https://jsperf.com/for-foreach-reduce](https://jsperf.com/for-foreach-reduce)<br />
![](https://images.gitee.com/uploads/images/2018/1209/181116_7e7cdfaa_1575229.png#align=left&display=inline&height=1207&originHeight=500&originWidth=500&status=done&style=none&width=1207)


还有forEach、for、reduce之间的性能差距：

![](https://images.gitee.com/uploads/images/2018/1209/182525_96336504_1575229.png#align=left&display=inline&height=1205&originHeight=500&originWidth=500&status=done&style=none&width=1205)

可以看到，虽然浏览器引擎已经对这些高阶函数进行了优化，但比起原生的for循环这些高阶函数的执行效率还是有所差别的~因此在使用函数式编程的时候还是需要考虑项目实际的需要，当然我个人认为只要不是对性能要求非常严格的情况都可以使用函数式编程来增加我们写的代码的可读性和可维护性。
