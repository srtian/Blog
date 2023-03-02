---


title: JavaScript数据处理之字符串<br />
categories: JavaScript<br />
tags:

- JavaScript

---


### 一、字符串处理

字符串在编程中占据着举足轻重的作用，我们通常会使用字符串来储存一些文字之类的信息，以便计算机能够对这些信息进行检索或理解。因此首先我们先来看看JavaScript是如何来对字符串进行处理的。


### 1.1 创建字符串

在JavaScript中有三种方法来创建字符串：

```javascript
const string1 = 'string 1'
const string2 = "string 2"
const string3 = `string 3`
```

前两种在JavaScript语法定义上并没有太大的区别，都可以更具自己的个人习惯或者团队的编码风格来自行选择。但第三个，也就是``就比较特殊了，虽然它也能实现一个普通的字符串，但通常来说，我们一般使用它来创建一个模板字符串本：

```javascript
// 模板字符串
const example = 'example'
const foo = `this is a ${example}`
console.log(foo) // this is a exmaple

// 也可以创建多行文本，前后两条语句的执行结果完全一致
console.log('string text line 1
' +
'string text line 2')
console.log(`string text line 1
string text line 2`)
```


### 1.2 字符串组合

上面就是我们在JavaScript中三种创建字符串的方式，但通常来说，我们常常需要将不同的信息通过各种方式进行拼装，从而产生一个我们所需要的字符串。比如我们上面所举的标签模板就是很常见的例子。

通常来说，最基本的字符串之间的连接我们可以直接通过+运算符来解决：

```javascript
const example = 'example'
const foo = 'this is a ' + example
conosle.log(foo) // this is a exmaple
```

但这种最基本的方式其实并不够优雅，且也只适用于数据量较小的字符串的拼接。若想数据量比较大时，需要将多个可变数据“嵌入”到一个模板中去，比如使用ES6的模板字符串来实现这个需求：

```javascript
const example = 'example'
const foo = `this is a ${example}`
console.log(foo) // this is a exmaple
```

或者是这种优雅的模板方式:

```javascript
const f00 = 'foo'
const bar = 'bar'
let url = `
    www.srtian96.gitee.io/blog/
    ?foo=${foo}
    &bar=${bar}
`
console.log(url) // www.srtian96.gitee.io/blog/?foo=foo&bar=bar
```


### 1.3 处理字符串

在创建完字符串后，我们所要做的就是对这些字符串来进行处理了。下面我在MDN上随便截取的一段话：

> As a group, the standards are known as Web Components. In the year 2018 it’s easy to think of Web Components as old news. Indeed, early versions of the standards have been around in one form or another in Chrome since 2014, and polyfills have been clumsily filling the gaps in other browsers.<br />
After some quality time in the standards committees, the Web Components standards were refined from their early form, now called version 0, to a more mature version 1 that is seeing implementation across all the major browsers. Firefox 63 added support for two of the tent pole standards, Custom Elements and Shadow DOM, so I figured it’s time to take a closer look at how you can play HTML inventor!


我们先来分析一下这一段话，中间充斥着大小写的字母，标点符号，以及空格。此外还需要考虑的是HELLO、hello 和 Hello 的意思其实都是一样的，所以我们需要先完成以下预处理任务：

- 去除标点符号及数字
- 将所有的大写字母转为小写


#### 1.3.1 去除标点符号及数字

在纯英文的情况下，去除标点符号其实本质上就是只保留英文字母。因此在进行筛选时，我们只需要直接筛选出英文字母和空格就行了。这里我们可以使用 ASCII 码进行甄别。大写字母的 ASCII 码范围为 65 到 90，而小写字母则为 97 到 122，空格的 ASCII 码为 32，换行符的 ASCII 码为 10。在 JavaScript 可以用 string.charCodeAt() 方法获取字符的 ASCII 码：

```javascript
const getText = (words) => {
    let word = words
    let text = ''
    for (let i = 0; i < word.length; i++) {
        const k = word[i]
        const asciiCode = k.charCodeAt()
        if ((asciiCode >= 65 && asciiCode <= 90) || (asciiCode >= 97 && asciiCode <= 122) || asciiCode === 32) {
            text += k
        }
    }
    return text
}
```

通过上面这个函数，我们就能够成功的将文本进行处理得到纯英语字母的文字了。


#### 1.3.2 大写转小写

前面我们在上面有提到，大小写的英语字母都有自己所对应的ASCII码，我们可以使用 string.charCodeAt() 方法来获取字符的 ASCII 码，同时JavaScript也提供给我们 String.fromCharCode()，来讲ASCII，码转化为对于的字符。而大写字母的ASCII码只要加上32就可以的达到对应的小写字母的ASCII码值了：

```javascript
const toBeLower = (words) => {
    let word = words
    let lower = ''
    for (let i = 0; i < words.length; ++i) {
        const k = words[i]
        const asciiCode = k.charCodeAt()
        if (asciiCode >= 65 && asciiCode <= 90) {
            lower += String.fromCharCode(asciiCode + 32)
            } else {
            lower += k
            } 
        }
    return lower
}
```

但其实在JavaScript中我们没必要这么麻烦的去通过更改ASCII码来讲大写字母转为小写字母，我们可以直接调用JavaScript所提供的API：string.toLowerCase(),所以我们想将一些文字进行处理，就可以这么做:

```javascript
const getPureText = (words) => {
    let word = words
    let text = ''
    for (let i = 0; i < word.length; i++) {
        const k = word[i]
        const asciiCode = k.charCodeAt()
        if ((asciiCode >= 65 && asciiCode <= 90) || (asciiCode >= 97 && asciiCode <= 122) || asciiCode === 32) {
            text += k
        }
    }
    return text.toLowerCase()
}
```

将我从MDN随意获取的那段文字传入这个函数中去，就可以得到这么一段已经被处理的文字：

> as a group the standards are known as web components in the year  its easy to think of web components as old news indeed early versions of the standards have been around in one form or another in chrome since  and polyfills have been clumsily filling the gaps in other browsers after some quality time in the standards committees the web components standards were refined from their early form now called version  to a more mature version  that is seeing implementation across all the major browsers firefox  added support for two of the tent pole standards custom elements and shadow dom so i figured its time to take a closer look at how you can play html inventor


当我们获得这么一段已经被处理的字符时，想要对其进行检索就变得更容易了，比如我想知道这段话到底有多少个单词：

```javascript
const words = '......' // 这是上面的那段话
const newWords = getPureText(words).split(' ')
console.log(newWords.length) // 123
```

我们可以很简单的得到这个结果。当然如果你对正则表达式足够熟悉的话，直接可以使用正则表达式来实现这个目的，代码非常简单：

```javascript
const getPureText = (words) => {
    const word = words.toLowerCase().match(/\w+/g)
    return word
}
console.log(getPureText(words).length)
```
