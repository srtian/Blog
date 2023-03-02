
# 一、脚本与模块
JavaScript中又两种源文件，这个区分从ES6开始：

- 脚本
- 模块

其中脚本可以由浏览器或者node环境引入执行，而模块只能由JavaScript 代码用 import 引入执行。

因此从概念上来讲，我们可以认为脚本是具有主动性的 JavaScript 代码段，是控制宿主完成一定任务的代码；而模块则是被动性的代码段，等待被调用的库。<br /> <br />现代的浏览器都支持使用script标签引入模块或者脚本，如果要引入模块，就必须给script标签添加 `type="module"` 。

脚本中可以包含语句，而模块则是由三部分组成：

- import声明
- export声明
- 语句


### import声明
import声明有两种使用方式

- 直接import一个模块
- 使用import form

直接import 一个模块，只能保证这个模块被执行，引用它的模块无法获得它的任何信息。

而 `import from` 则是引入模块中的一部分信息，可以将它们变成本地变量。他有三种用法：
```javascript
import x from "./a.js" // 引入模块中导出的默认值。
import {a as x, modify} from "./a.js"; // 引入模块中的变量。
import * as x from "./a.js" // 把模块中所有的变量以类似对象属性的方式引入
// 第一种方式还可以和后两种组合使用
import d, {a as x, modify} from "./a.js"
import d, * as x from "./a.js"
```
需要注意的是使用没使用 as 的默认值永远在最前，且这里的as只是换一个名字而已，当变量被改变的时候，as所产生的值也会随着改变。


### export


