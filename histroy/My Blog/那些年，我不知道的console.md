在日常的代码调试中，console.log算是我们最常用的调试方法之一了。但对于console本身除了log外，还有哪些方法我并不是很清楚，因此利用了一下平时的空余时间，翻阅了一下MDN的问档，将console比较常用的方法过了一遍，算是增加一下自己调试代码的手段。

### 一、console.log({value})
在日常的打log中，我们经常使用诸如下面的方式来打印我们想要打印的数据：
```javascript
console.log('value', value)
console.log('data', data)
```
但其实根本就无需这么麻烦，因为 ES6 中引入了`enhanced object literal(增强对象文字面量)，`我们可以直接使用 console.log({value}) 的方式来打印出我们想要打印的数据，这样既可以提高log的可读性，也可以节省不少我们为log命名的时间，一举两得啊：<br />                                                      ![console.PNG](https://cdn.nlark.com/yuque/0/2019/png/296173/1570545775050-defb4ca6-e6eb-42ac-9763-3f3e4448b3ad.png#align=left&display=inline&height=287&name=console.PNG&originHeight=287&originWidth=300&size=7865&status=done&width=300)

### 二、console.table()
除此之外，让我感到收获最多的就是console.table()了。console.table 适用于我们想要打印一个数组或者对象。它不仅可以更具数组中包含的对象的所有数学去计算出表的列名，而且这些列还是可以缩放甚至排序的。如图：<br />**![table.PNG](https://cdn.nlark.com/yuque/0/2019/png/296173/1570546569309-d80df0b1-a635-4b25-8f66-f1a75e416b08.png#align=left&display=inline&height=274&name=table.PNG&originHeight=274&originWidth=891&size=11352&status=done&width=891)**<br />**<br />当我们需要打印多个数组或者对象的时候，我们也可以使用 console.table({value}) 来帮助我们搞事情，这个是由我们变量名称就会变成我们第一列的 index 了：<br />![table1.PNG](https://cdn.nlark.com/yuque/0/2019/png/296173/1570546876414-214a16f7-6dad-4862-b27e-275c804c015b.png#align=left&display=inline&height=472&name=table1.PNG&originHeight=472&originWidth=996&size=19487&status=done&width=996)

是不是相当的花里胡哨的！


### 三、console.trace()
console.trace() 可以打印出函数的调用栈的信息，比如这样：<br />![企业微信截图_15705191027838.png](https://cdn.nlark.com/yuque/0/2019/png/296173/1570535953058-3d6d50fd-13ef-4f6f-b9b5-747ab1264d1e.png#align=left&display=inline&height=489&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15705191027838.png&originHeight=489&originWidth=587&size=40527&status=done&width=587)

### 四、console.time() and console.timeEnd()
console.time()和console.timeEnd()，可以用来显示代码的运行时间<br />![企业微信截图_15706245177257.png](https://cdn.nlark.com/yuque/0/2019/png/296173/1570624567930-8630aa9e-0645-4b27-bcc8-d9f17cd782ba.png#align=left&display=inline&height=205&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15706245177257.png&originHeight=205&originWidth=675&size=9017&status=done&width=675)

### 五、console.warn()
console.warn()简单来说就是对console.log最好的替换，它在功能上其实是和console.log()其实是一样的，但它还能对打印的东西进行凸显，具体表现如下：<br />![企业微信截图_15706250828024.png](https://cdn.nlark.com/yuque/0/2019/png/296173/1570625129666-088e5862-4c27-44d4-90ef-45365e6b3cf0.png#align=left&display=inline&height=183&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15706250828024.png&originHeight=183&originWidth=784&size=7288&status=done&width=784)

### 六、console.group()
当我们由很多console.log的时候我们就可以通过console.group()和console.groupEnd()来对这些打印进行分组，让打印信息显得更有条理：

### ![企业微信截图_1570626318254.png](https://cdn.nlark.com/yuque/0/2019/png/296173/1570626363886-d2c69329-531a-4d13-90ae-94c627afb88c.png#align=left&display=inline&height=439&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1570626318254.png&originHeight=439&originWidth=791&size=22930&status=done&width=791)

