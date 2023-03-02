
### 前言
用了TypeScript挺久了，工具链这边显著的特点就是不少模块由于业务原因亦或者是出于其他的一些考虑，后端所传输过来的数据很复杂，因此处理起来需要小心翼翼的，稍有不慎就会出现TypeError，因此之前在团队内部有做过一个小的分享，即如何使用TS让自己的数据处理更安全。这里将之前的PPT内容写成文章，此外也去除一些与具体业务有关的东西，方便后续补充以及沉淀～（话说PPT迷之找不到了，只剩下一些截图来和思维导图了）

#### 思维导图
![](https://cdn.nlark.com/yuque/0/2020/svg/296173/1589540812033-d9479ca5-0ca1-4a40-a22f-dd9934a0f7dc.svg)
## 1.关于keyof，小东西有大能量
```javascript
interface IDataSet {
  name: string
  id: number
  type: string
  sampleImages: {
    imagePath: ''
    list: []
    algoType: ''
  }
}

const transformData = (data: IProps) => {
    Object.keys(data).map(item => {
        if(item === 'sampleImege') {
            // ...做些小操作
        }
      // ...code
    })
}
```
上面一段代码乍一看没啥问题，但其实永远也无法执行到小操作那里去，因为  `sampleImages`  这个单词我们错误拼写成了 `sampleImege` ，而类似的拼写错误也正是我们经常容易所忽视的，因此正确的做法是使用 `keyof` ，来保护我们的粗心大意：
```javascript
type dataset = keyof IDataSet

const transformData = (data: IDataSet) => {
    Object.keys(data).map((item: dataset) => {
        if(item === 'sampleImage') {
            return 'haha'
        }
    })
}
```
这样就可以写代码的时候获得相关提示：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/296173/1589529553178-51c76a2f-39e9-4c93-991b-94435859fa60.png#align=left&display=inline&height=176&name=image.png&originHeight=528&originWidth=1738&size=81204&status=done&style=none&width=579.3333333333334)<br />（实际上在使用vscode的时候，只要打出了那个空字符串，就会提示有哪些可选的参数了，完全不用自己再去打一遍，十分方便快捷且安全）

同时，在获取对象的值的时候，我们也可以使用keyof来保护我们的代码，还是直接是用我们上面的 `IDataSet` 为例，这样写我们完成发现不了问题:
```javascript
const getData = (data: IProps) => {
  if(data['ids'] === 10086) {
  	...
  }
}
```
这个时候我们就可以使用 由`keyof` 所实现的 `get` 来武装我们的代码，让我们的代码变得更安全：
```javascript
function get<T extends object, K extends keyof T>(o: T, name: K): T[K] {
  return o[name]
}
const getData = (data: IDataSet) => {
  if(get(data, 'ids')) {
    // ...code
  }
}
```
这样当我们不小心将key写错的时候，就会有相应的提示来:![image.png](https://cdn.nlark.com/yuque/0/2020/png/296173/1589532010316-cd8cf056-4ba1-41ba-841f-ee50bc0c7b4b.png#align=left&display=inline&height=119&name=image.png&originHeight=358&originWidth=1770&size=76948&status=done&style=none&width=590)<br />// 待续
