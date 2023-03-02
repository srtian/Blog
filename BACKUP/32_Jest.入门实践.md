# [Jest 入门实践](https://github.com/srtian/Blog/issues/32)


# 一、基础
前端自动化测试所带来的收益：

- 杜绝由于各种疏忽而引起的功能Bug
- 快速反馈，例如：对于UI组件，可以让脚本代替手动点击
- 有利于多人协作

前端现今主流测试框架：

- MOCHA：是一个功能丰富的JS测试框架，运行在Node.js和浏览器中，使异步测试变得简单有趣。
- Jest：由FB开源的前端测试框架，也是我们公司现在所使用的测试框架。

Jest前端测试框架的优点：

- 新：前端娱乐圈了解一下
- 基础好，出身好：FB出品
- 速度快：具有单独模块测试功能
- API简单
- 隔离性好
- IDE配合：VSCode
- 支持多项目并行
- 快出覆盖率


# 二、Jest的使用实践

## 2.1、基础实践
Jest的下载非常简单，只需要在本地由Node.js的运行环境，然后就可以使用npm包来对Jest进行下载来：
```javascript
> mkdir learnJest
> npm init
> npm install jest@24.8.0 -D
```
这里的-D就是保存到dev里边，而在线上就不使用它。

然后我们就可以在我们的基础目录下面新建一个叫做 `demo.js` 的文件以及一个叫做 demo.test.js的文件，然后敲些代码：<br />
```javascript
// demo.js
function add(a, b){
    return a + b
}

function menus(a, b){
    return a - b
}
module.exports = {
    add,
  	menus
}

// demo.test.js
const demo = require("./demo.js");
const { add, menus } = demo;

test("测试add", () => {
  expect(add(2, 5)).toBe(7);
});
test("测试add", () => {
  expect(menus(3, 2)).toBe(1);
});
```
至此，我们就将我们的代码以及测试用例给完成了。没有使用过Jest的同学可能到这里就会要问，上面这种语法所代表的含义是什么呀？别慌，其实 `expect` 和 `test` 的内部逻辑大致是这样的：
```javascript
const expect = result => {
  return {
    toBe: function(actual) {
      if (result !== actual) {
        throw new Error(`没有通过测试, 预期${actual}, 结果为${result}`)
      }
    }
  }
}

const test = (desc, fn) => {
  try {
    fn()
    console.log(`${desc} 通过测试`)
  } catch(e) {
    console.error(`${desc}没有通过测试, ${e}`)
  }
}
```
接下来，我们就可以进行测试了。那么我们如何进行测试呢，其实也很简单，我们只需要将package.json 文件。将里面的 `scripts` 标签的值改一下就行了：
```json
{
  "name": "jesttest",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "jest"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "jest": "^24.8.0"
  }
}
```
然后我们在终端中执行 `npm run test` 即可：
```bash
> npm run test

> jeststydy@1.0.0 test /Users/ruotian.shen/My-Study/jestStydy
> jest

 PASS  src/demo.test.js
  ✓ 测试add (2ms)
  ✓ 测试add
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        1.468s
Ran all test suites.
```

## 2.2、基础配置
我们可以使用来生成默认的 `jest`  文件：
```bash
> npx jest --init
```
然后需要回答三个问题，最后就可以生成一个叫做 `jest.config.js` 的配置文件。这个配置文件里面就可以去配置一些关于 Jest 的相关东西。其中有一项 coverageDirectory 的东西，就和我们经常听到的代码测试覆盖率有关。

所谓的代码测试覆盖率，就是我们的测试代码，对功能性代码和业务逻辑代码作了百分多少的测试，这个百分比，就叫做代码测试覆盖率。如果我们开启了这一项：<br />
```json
coverageDirectory : "coverage" 
```
我们就可以生成对应的覆盖率：
```bash
> npx jest --coverage
 PASS  src/demo.test.js
  ✓ 测试add (2ms)
  ✓ 测试add (1ms)

----------|----------|----------|----------|----------|-------------------|
File      |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
----------|----------|----------|----------|----------|-------------------|
All files |      100 |      100 |      100 |      100 |                   |
 demo.js  |      100 |      100 |      100 |      100 |                   |
----------|----------|----------|----------|----------|-------------------|
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        1.715s
Ran all test suites.
```
除了生成一个终端的报表。其中：

-  `% Stmts`  表示的是语句覆盖率，即语句的测试覆盖范围
- `% Branch` 表示的是分支覆盖率，即if代码块测试覆盖范围
- `% Funcs` 表示的是函数覆盖率，即函数的测试覆盖范围
- `% Lines` 表示的是行覆盖率，即代码行数的测试覆盖范围

这条命令还能会生成一个叫做 `coverage` 的文件夹，里面有个 `index.html` 的文文件，这里就有一个花里胡哨的页面，也可以展示对应的代码覆盖率：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/296173/1589962550737-d2e5f8d8-fde0-49f8-986c-281d19319a39.png#align=left&display=inline&height=185&name=image.png&originHeight=556&originWidth=2774&size=80207&status=done&style=none&width=924.6666666666666)<br />此外，Jest默认支持的是 `CommonJS` 的语法，目前还不支持 `import form` 的语法，使用的时候会报错。如果我们想要使用ES6的语法，就需要使用babel来代码转成 `CommonJS` 代码。
```bash
npm install @babel/core@7.4.5 @babel/preset-env@7.4.5 -D
```
然后我们新建一个 `babelrc` 的文件：
```json
{
    "presets":[
        [
                "@babel/preset-env",{
                "targets":{
                    "node":"current"
                }
            }
        ]
    ]
}
```
这样我们就可以愉快的使用使用ES6的语法来用Jest了。

## 2.3、Jest的匹配器
Jest匹配器在Jest中，可以说是最重要的功能之一，前面我们说的 `toBe()` 就是匹配器的一种，Jest 使用“匹配器”让你使用不同方式测试数值。

### 1、toBe
toBe()适配器，可以说是Jest中最常用的适配器，我们可以简单的将其理解为**严格相等**也就是我们常用的 `===` 。
```bash
test('two plus two is four', () => {
  expect(2 + 2).toBe(4);
});
```

### 2、toEqual
toBe和toEqual的区别就在与，toBe是严格相等的判断，而 toEqual 则是只要内容相等即可，比如这样：
```javascript
test('测试严格相等',()=>{
    const a = {name:'srtian'}   
    expect(a).toBe({name:'srtian'})
})
test('测试内容相等',()=>{
    const a = {name:'srtian'}   
    expect(a).toEqual({name:'srtian'})
}) 
```
然后我们执行一下test，就可以发现这两个API之间的区别：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/296173/1589964776202-ad3d5e3f-22d2-4c40-8ce6-eb159d909eba.png#align=left&display=inline&height=354&name=image.png&originHeight=1062&originWidth=1482&size=193644&status=done&style=none&width=494)

### 3、Truthiness
有时候，我们需要来判断匹配null、undefined、false。Jest也提供了相对应的 API 来进行匹配：

- `toBeNull` 只匹配 `null`
- `toBeUndefined` 只匹配 `undefined`
- `toBeDefined` 是 `toBeUndefined` 反义词
- `toBeTruthy` 匹配任何 `if` 语句当作真值的表达式
- `toBeFalsy` 匹配任何 `if` 语句当作假值的表达式
```javascript
test('null', () => {
  const n = null
  expect(n).toBeNull()
  expect(n).toBeDefined()
  expect(n).not.toBeUndefined()
  expect(n).not.toBeTruthy()
  expect(n).toBeFalsy()
})

test('zero', () => {
  const z = 0
  expect(z).not.toBeNull()
  expect(z).toBeDefined()
  expect(z).not.toBeUndefined()
  expect(z).not.toBeTruthy()
  expect(z).toBeFalsy()
})
```

### 4、Number匹配
Jest针对 `number` 也提供了相关的api来进行匹配：

- toBeGreaterThan
- toBeLessThan
- toBeGreaterThan
- toBeGreaterThanOrEqual
- toBeLessThanOrEqual
- toBeCloseTo：这个是可以自动消除 `JavaScript`浮点精度错误的匹配器



前面几个匹配器，看名字就非常直观，而最后一个匹配器 `toBeCloseTo` 是一个可以自动清除浮动 `JavaScript` 浮点精度错误的匹配器
```javascript
test('toEqual匹配器',()=>{
    const a = 0.1
    const b = 0.2
    expect(a + b).toEqual(0.3)
}) // 不会通过测试用例

test('toBeCloseTo匹配器',()=>{
    const c = 0.1
    const d = 0.2
    expect(c + d).toBeCloseTo(0.3)
}) // 可以通过测试用例
```

## 2.4、测试异步代码
当我们使用Jest来测试异步代码时，Jest需要知道当前测试的代码是否已经完成，若已经完成，它就可以转移到另一个测试。

### 1. 回调
```javascript
test('the data is peanut butter', done => {
  function callback(data) {
    try {
      expect(data).toBe('peanut butter');
      done();
    } catch (error) {
      done(error);
    }
  }

  fetchData(callback);
});
```

### 2. Promise
如果fetchData 不使用回调函数，而是返回一个Promise，我们可以这样测试它：
```javascript
test('the data is peanut butter', () => {
  return fetchData().then(data => {
    expect(data).toBe('peanut butter');
  });
});
```
需要注意的是，我们要将 `Promise` 作为 `return` 的值。如果不这样做的话，在 `fetchData` 返回的这个 `Promise` 被 `resolve` 和 `then` 执行结束之前，测试就已经被视为完成了。

如果我们向匹配 `Promise`  的状态为 `rejected` ，我们可以使用 `catch`  方法来捕获对应的错误信息。需要注意的是 要确保使用 `expect.assertions` 来验证一定数量的断言被调用。否则一个 fulfilled 状态的 Promise 不会让测试失败：
```javascript
test('the fetch fails with an error', () => {
  expect.assertions(1);
  return fetchData().catch(e => expect(e).toMatch('error'));
});
```
我们也可以使用 expect 语句中使用 `resolves` 匹配器， `Jest` 将等待此 `Promise` 解决。如果承诺被拒绝，则测试将自动失败：
```javascript
// 匹配成功
test('the data is peanut butter', () => {
  return expect(fetchData()).resolves.toBe('peanut butter');
});
// 匹配失败            
test('the fetch fails with an error', () => {
  return expect(fetchData()).rejects.toMatch('error');
});
```

### 3. async await
既然可以使用Promise 来进行测试，那么使用async也同理可以了，并且写起来也很直观简单：
```javascript
test('the data is peanut butter', async () => {
  const data = await fetchData();
  expect(data).toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  expect.assertions(1);
  try {
    await fetchData();
  } catch (e) {
    expect(e).toMatch('error');
  }
});
```
也可以这样：
```javascript
test('the data is peanut butter', async () => {
  await expect(fetchData()).resolves.toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  await expect(fetchData()).rejects.toThrow('error');
});
```

## 2.5、Jest中的钩子函数
Jest中也提供了4个钩子函数，来帮助我们在执行测试期间去进行一些操作：

- beforeAll()：运行在所有测试开始之前
- afterAll()：运行在所有测试完成之后
- beforeEach()：运行在每个测试开始之前
- afterEach()：运行在每个测试完成之后

具体例子：
```javascript
import { add, menus } from "./demo";

beforeAll(() => {
  console.log("这个会最先执行");
});

beforeEach(() => {
  console.log("这个会执行两次");
});
test("test add", () => {
  expect(add(1, 2)).toBe(3);
});
test("test menus", () => {
  expect(menus(3, 1)).toBe(2);
});

afterEach(() => {
  console.log("这个也会执行两次");
});

afterAll(() => {
  console.log("这个会最后执行");
});

```
运行 `npm run test` ，即可得到以下的结果:<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/296173/1590738307693-5d8f0642-62fe-4a21-b77c-61461b208de9.png#align=left&display=inline&height=313&name=image.png&originHeight=938&originWidth=1192&size=271035&status=done&style=none&width=397.3333333333333)

## 2.6、Jest中的Mock
