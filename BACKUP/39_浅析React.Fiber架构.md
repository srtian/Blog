# [浅析React Fiber架构](https://github.com/srtian/Blog/issues/39)

做了一次React Fiber架构的分享[Fiber架构分享.pptx](https://www.yuque.com/attachments/yuque/0/2020/pptx/296173/1594286402740-29fda8e6-e97b-4a60-8840-b46a90e7b24c.pptx?_lake_card=%7B%22uid%22%3A%221594286400762-0%22%2C%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2020%2Fpptx%2F296173%2F1594286402740-29fda8e6-e97b-4a60-8840-b46a90e7b24c.pptx%22%2C%22name%22%3A%22Fiber%E6%9E%B6%E6%9E%84%E5%88%86%E4%BA%AB.pptx%22%2C%22size%22%3A849232%2C%22type%22%3A%22application%2Fvnd.openxmlformats-officedocument.presentationml.presentation%22%2C%22ext%22%3A%22pptx%22%2C%22progress%22%3A%7B%22percent%22%3A99%7D%2C%22status%22%3A%22done%22%2C%22percent%22%3A0%2C%22id%22%3A%22saP1U%22%2C%22card%22%3A%22file%22%7D)，这是文字版的。

# 一、.为什么会出现 Fiber 架构
React 15 至 React 16 发生的一个主要的变化是，从原先的 Stack Reconciler 转为了 Fiber Reconciler。那为什么React 团队要进行这样的更改呢？先让我们看一下 Stack Reconciler 所存在的问题。

在Stack Reconciler 中，进行渲染时，父组件调用子组件，我们可以类比为函数递归。当组件进行 diff 后，会以递归的方式去深度优先的遍历整个组件树，从而达到将整个组件树全部计算和渲染的目的：<br />                                             ![image.png](https://cdn.nlark.com/yuque/0/2020/png/296173/1596771399462-5b17d68b-de05-4ebb-9d5e-466da8096dcd.png#align=left&display=inline&height=189&name=image.png&originHeight=568&originWidth=902&size=165663&status=done&style=none&width=300.6666666666667)<br />但这样做就会有一个问题，我们都知道浏览器给予用户一个不卡顿的体验，一般需要保证1秒60祯的刷新频率，换算下来，就是浏览器的每一帧生成的时间应该是在 16ms 左右。而大部分垃圾时间（janky），我们都可以归类于：同步任务所占用的CPU时间（主要是 JS 运算）+ 原始DOM更新。而Fiber的出现，主要所要解决的就是同步任务所占用的 CPU 时间（JS运算等等），其解决的方式是，将一个大块的同步任务（也就是我们上面所说的递归的去diff和计算虚拟DOM树），拆分成一个个小的同步任务，然后在每一个时间切片之间，去判断是否存在一些优先级更高的的事情，如果存在就优先去处理优先级高的任务，从而保证一次渲染计算不会占用太长的CPU运算的时间，让后续的 layout 和 paint 的时间是足够的：<br />                                              ![image.png](https://cdn.nlark.com/yuque/0/2020/png/296173/1596772363767-7e468264-489b-4640-9b46-9441d9d83357.png#align=left&display=inline&height=196&name=image.png&originHeight=588&originWidth=1042&size=575353&status=done&style=none&width=347.3333333333333)

# 二、Fiber 架构的具体实现
对于Fiber架构的整体构成，我个人理解，主要可以分为以下两个部分：

- Fiber调度算法
- Fiber节点

其中Fiber调度算法，主要是用于任务优先级的确认（即高优先级的任务先指向，低优先级的任务的恢复亦或者直接放弃） 。而Fiber节点，则是一种数据结构，我们可以理解为一个对象，它是有虚拟DOM生成，用于存储一个渲染单位的一些基本信息，是Fiber实现的基石，其主要属性如下：
```javascript
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
 // 静态数据结构属性
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;
 	this.ref = null;
  // 位置属性
  this.return = null;
  this.child = null;
  this.sibling = null;

  // 状态属性
  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;
 // 记录 Effect 的属性
  this.effectTag = NoEffect;
  this.nextEffect = null;
  this.firstEffect = null;
  this.lastEffect = null;

  // 调度优化优先级
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  // 用于实现双缓存的属性
  this.alternate = null;
}
```
React Fiber下的组件创建与更新，其本质上就是去构建一个由多个Fiber 节点相连组成的 Fiber 节点树的过程。而创建和更新 Fiber 节点树，React又将其分为了两个阶段去完成：

- Render 阶段（或者叫做：Reconciler 阶段）
- Commit阶段

## 2.1、Render阶段
React在Render阶段，主要做的一个事情是生成一个用于更新的 Fiber节点树，而在这个过程中，React不会做任何有副作用的事情，只会对副作用去做一个收集，等到Commit阶段再去做。这主要是由于我们上面所说的，React 的更新不再是一次性完成的了，而是可能会进行多次反复横跳，也就是说在Render阶段做的事情，可能会由于优先级的问题被执行多次。而具有副作用的一些操作，通常不是幂等的，如果被执行多次，可能会产生开发者预料不到的问题。而Commit阶段，则不会执行多次，因此React会将副作用操作进行收集，统一到Commit去完成，具体实现步骤可以看如下的思维导图：
![](https://cdn.nlark.com/yuque/0/2020/png/296173/1596792033804-a468c576-94b6-40ba-8ea5-52903c029fa9.png)
具体总结就是在Render阶段会做两个事情：

1. 进行BeginWork，如果有Current Fiber Tree，则会依据Current Tree进行DIFF，生成 WorkInProgress 树，如果没有，则直接进行深度优先遍历去生成 WorkInProgress
2. 进行Complete Work，对副作用进行收集，组成Effect List

## 2.2、Commit阶段<br />
在得到最后所需要渲染的 WorkInProgress 树后，React会进入Commit阶段，在这个阶段React主要会做以下一些事情：

- 执行Render之后的生命周期函数
- 执行副作用操作

而它内部将Commit也分成了三个不同的阶段：

1. before mutation阶段（执行DOM操作前）
2. mutation阶段（执行DOM操作）
3. layout阶段（执行DOM操作后）

主要做的事情如下思维导图：
![](https://cdn.nlark.com/yuque/0/2020/png/296173/1596792033816-d32d522a-4148-4524-bbee-6faf250e1e15.png)（这里有一个小的Tip需要注意，Hook中的 useEffect 是在Commit阶段之后执行的，也就是说，实际上它调用的时机会比所有的生命周期函数都要晚）

# 三、Fiber的局限性
首先就是Fiber的实现过于复杂，导致代码量以及源码的复杂度都直线上升，一方面增加了React的包的大小（相较于Vue），另一方面是让React源码的复杂度提升，更具有黑盒效益了。

另一方是，React Fiber所带来的收益往往没有我们想的那么大。具体的一些论述，可以去看尤大在Github上的一个回答，很精彩：
> [https://github.com/vuejs/rfcs/issues/89](https://github.com/vuejs/rfcs/issues/89)

![image.png](https://cdn.nlark.com/yuque/0/2020/png/296173/1596791623128-bedd3097-eadb-4053-a029-fb91dc434118.png#align=left&display=inline&height=708&name=image.png&originHeight=1416&originWidth=1504&size=888327&status=done&style=none&width=752)

参考资料：

- [https://github.com/vuejs/rfcs/issues/89](https://github.com/vuejs/rfcs/issues/89)
- [https://zhuanlan.zhihu.com/p/60307571](https://zhuanlan.zhihu.com/p/60307571)
- [https://segmentfault.com/a/1190000020736966](https://segmentfault.com/a/1190000020736966)
