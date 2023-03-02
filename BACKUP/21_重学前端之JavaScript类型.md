# [重学前端之JavaScript类型](https://github.com/srtian/Blog/issues/21)

按照JavaScript最新标准，一共定义了7种数据类型：

1. Undefined；
2. Null；
3. Boolean；
4. String；
5. Number；
6. Symbol；
7. Object；


### 一、七大数据类型

#### 1、Undefined和Null；
区别：

- Undefined是一个全局变量，而 null 则是一个关键字
- 在语义上来讲，null表示为：“定义了，但是为空”；而Undefined则表示任何变量在复制前都是undefined类型的。



#### 2、Boolean
只有两个值，true和false，表示逻辑上的真和假


#### 3、String
String用于文本数据，最大长度为2^53-1。需要注意的是String的意义并非字符串，而是字符串的UTF16编码，因此字符串的方法都是针对UTF16编码的。

此外，JavaScript的字符串是永远无法变更的，一旦字符串构造出来，就不能以任何形式的方式改变字符串的内容，所以字符串具有值类型的特征。（这个和Java类似）


#### 4、Number
Number对应的就是数字，大致对应数学中的有理数。JavaScript的Number类型有18437736874454810627(即 2^64-2^53+3)个值。

Number中有三个需要注意的值：

- NaN，占用了 9007199254740990，这原本是符合 IEEE 规则的数字；
- Infinity，无穷大；
- -Infinity，负无穷大。

一段经典的代码：
```javascript
console.log( 0.1 + 0.2 == 0.3);
```
其原因就是因为浮点数运算的精度问题所导致的。正确的运算方法应该是：
```javascript
console.log( Math.abs(0.1 + 0.2 - 0.3) <= Number.EPSILON);
```

#### 5. Symbol
Symbol 是 ES6 中引入的新类型，它是一切非字符串的对象 key 的集合，在 ES6 规范中，整个对象系统被用 Symbol 重塑。Symbol 可以具有字符串类型的描述，但是即使描述相同，Symbol 也不相等


#### 6. Object
JavaScript中，对象的定义是“属性的集合”。其中属性有分为：数据属性和访问器属性，二者都是key-value的结构，key可以是字符串和Symbol。<br />JavaScript 中的几个基本类型，都在对象类型中有一个“亲戚”。它们是：

- Number；
- String；
- Boolean；
- Symbol。

需要注意的是， 3 与 new Number(3)是完全不一样的，一个是Number类型一个是对象类型。而Number、String 和 Boolean这三个构造器也是两用的，当和new搭配时，会产生对象，当直接调用时，会进行强制类型转换。

