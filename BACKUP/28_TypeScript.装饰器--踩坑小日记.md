# [TypeScript 装饰器--踩坑小日记](https://github.com/srtian/Blog/issues/28)


### 前言
最近在思考如何实现一个类型错误拦截的功能，意外的发现了基于装饰器实现的这个:
> [https://www.npmjs.com/package/reflect-metadata](https://www.npmjs.com/package/reflect-metadata)

加之之前实现国际化以及之前用 python的时候也使用到了装饰器，因此利用闲余时间学习了学习装饰器的一些特质以及使用技巧，越发感受到了装饰器功能的强大，写上这么一篇文章作为总结与记录，不过鉴于JS装饰器的语法介绍文章已经很多了，因此在此就不多做赘述了，只将TS中使用装饰器时需要注意的总结一下。


# 一、何为装饰器
关于装饰器模式的定义，我看到一个比较好的定义是：
> **Design Patterns** - **Decorator Pattern**. **Decorator pattern** allows a user to add new functionality to an existing object without altering its structure

即可以在不改变原有结构的基础上添加新功能。而这种编程方式也有一个编程范式与之相对应：面向切面编程（AOP）。AOP允许我们分离横切关注点，以此达到增加模块化程度的目标，它可以在不修改代码自身的前提下，给已有代码增加额外的行为。

其优点在于：装饰器和被装饰类可以独立发展，不会相互耦合，装饰模式是继承的一个替代模式，装饰模式可以动态扩展一个实现类的功能。


# 二、在TavaScript中的实践。

## 2.1、TypeScript中装饰器的基本使用
其实在JS中，装饰器本质上就是一个函数，他可以接受一些参数，并对这些参数进行修改，从而达到对被装饰对象进行赋能的目的。举个简单的栗子，我在下方声明了一个很简单的装饰器，其作用是打印一个字符串以及该class的constructor：
```javascript
import * as React from "react";
import "./styles.css";
const decoratorDemo = (constructor: any) => {
  console.log("it is a demo by decorator", constructor);
};
@decoratorDemo
export default class App extends React.Component {
  render() {
    return (
      <div className="App">
        <h1>Hello CodeSandbox</h1>
        <h2>Start editing to see some magic happen!</h2>
      </div>
    );
  }
}

// 输出：
// it is a demo by decorator 
// function App() {}

```
在TypeScript中使用装饰器并无太多坑，但以下几点需要注意：

- 装饰器是在被装饰的类创建好之后立即去进行修饰的，与这个类被调用了多少次无关。（比如我们上面所举出的栗子，虽然 `<App />` 被调用了三次，但装饰器函数之被调用了一次）
- 允许多个装饰器同时去修饰一个类，但它会从下往上依次执行，具体也可以去看我上面所举的那个链接。


## 2.2、干掉Any
上面我们在进行装饰器的定义时，对于 `constructor` 使用了any类型，这显然不符合我们使用TS的初衷：类型提示以及类型检查。因此我们在TS中使用装饰器，第二步要做的就是将这个 `any` 类型给干掉，代码如下：
```javascript
type IConstructor = new (...args: any[]) => any;

const decoratorDemo1 = <T extends IConstructor>(constructor: T) => {
  return class extends constructor {
    getName() {
      console.log(this.name);
    }
    name = "srtian";
  };
};

@decoratorDemo1
class Demo {
  name = "Bob";
}


```
如此这般我们就可以愉快享受TS给予我们的提示了：
```javascript
const decoratorDemo = <T extends IConstructor>(constructor: T) => {
  console.log("it is a demo by decorator", constructor);
  console.log(constructor.prototype); // -> 写这里的时候就会有如下的提示了
};
```
![image.png](https://cdn.nlark.com/yuque/0/2020/png/296173/1589020333962-b65c40b3-118e-422f-81e5-3d3aa845100d.png#align=left&display=inline&height=173&name=image.png&originHeight=518&originWidth=1074&size=68840&status=done&style=none&width=358)


## 2.3、解决类型检查
好了，咱们现在已经解决了，定义装饰器时，所需要定义的类型。但这里还有一个问题，当我们在使用它的时候还是会报错，比如我们这样写：
```javascript
const decoratorDemo1 = <T extends IConstructor>(constructor: T) => {
  return class extends constructor {
    getName() {
      console.log(this.name);
    }
    name = "srtian";
  };
};
@decoratorDemo1
class Demo {
  name = "Bob";
}
const demo = new Demo();
demo.getName() // <-这里会报错，如下
```
![image.png](https://cdn.nlark.com/yuque/0/2020/png/296173/1589020966210-9b7ccb28-5446-4b2b-9ce9-2120ab7e4f6b.png#align=left&display=inline&height=73&name=image.png&originHeight=218&originWidth=1100&size=32883&status=done&style=none&width=366.6666666666667)<br />遇到这种问题，我们就需要使用函数柯里化来帮助我们解决这个任务：
```javascript
const decoratorDemo2 = () =>
  function<T extends IConstructor>(constructor: T) {
    console.log("decorator Demo1");
    return class extends constructor {
      getName() {
        console.log(this.name);
      }
      name = "srtian";
    };
  };
const demo3 = new demo2()
 demo3.getName()
```
![image.png](https://cdn.nlark.com/yuque/0/2020/png/296173/1589021223372-876e3255-10f3-49fa-8763-66d41b02ee54.png#align=left&display=inline&height=71&name=image.png&originHeight=214&originWidth=1056&size=32897&status=done&style=none&width=352)<br />这样我们就可以愉快的使用装饰器来修饰我们的类了

所有代码链接：
> [https://codesandbox.io/s/affectionate-darwin-5zmuj?file=/src/App.tsx](https://codesandbox.io/s/affectionate-darwin-5zmuj?file=/src/App.tsx)


