# [五分钟，简单聊一聊React Component的发展历程](https://github.com/srtian/Blog/issues/4)

### 一、 前言
随着 react 最新的一个大版本中，给我们带来了 Hooks：[React v16.8: The One With Hooks](https://reactjs.org/blog/2019/02/06/react-v16.8.0.html)，从而将 Function component 的能力提高了一大截，成功的拥有了可以与 Class component 抗衡的能力。但话说回来，虽然 Hooks 看起来很美好，最近也有不少文章都讲解了Hooks这一“黑魔法”，但技术的不断演进，本身就是一个解决以往所存在问题的过程，因此我个人认为着眼于现在，回望过去，去看一看 react component 的发展之路，去看看 Class component 以及 Function component 为什么会出现以及它们出现的意义，所要解决的问题，也对于我们全面了解 react 是很有帮助的。

从 react component 的发展历程上来看，它主要是经历了一下三个阶段：
1. createClass Component 
2. Class Component 
3. Function Component

这个三个阶段也是react的组件不断走向轻量级的一个过程。其中 Class Component 完全替代了 createClass Component 成为了现在我们开发 react 组件的主流，而 Function Component 也在 Hooks 推出后磨刀霍霍，准备大干一场。下面就让我们去看看三者的具体情况吧~
> 注：这篇文章整体只是对React Component的发展历程的一个概括或者说是我自己学习后的一个整理，想要详细了解，还请看看我在文章贴的那些链接。

### 二、 createClass Component 
说实话，createClass Component 我也没用过，因为我接触到 react 的时候已经是2017年下半年了，那时候 ES6 已经大行其道，class component 也已经完全取代了 createClass Component。但现在看来 createClass Component 的语法也很简单，并不复杂：

```JavaScript
import React from 'react'

const MyComponent = React.createClass({
  // 通过proTypes对象和getDefaultProps()方法来设置和获取props
  propTypes: {
    name: React.PropTypes.string
  },
  getDefaultProps() {
    return {

    }
  },
  // 通过getInitialState()方法返回一个包含初始值的对象
  getInitialState(){ 
        return {
            sayHello: 'Hello Srtian'
        }
    }
  render() {
    return (
      <p></p>
    )
  }
})

export default MyComponent
```

react.createClass的语法并不复杂，它通过 createClass 来创建一个组件，并通过propTypes和getDefaultProps来获取props，通过通过getInitialState()方法返回一个包含初始值的对象，虽然从现在看来还是有点麻烦，但总体上来看代码也比较清晰，跟现在的 Class Component差别并不是太大。但 react.createClass 自从 react 15.5版本就不再为 react 官方所推介，而是想让大家的使用 class component 来代替它。而且在 react 16版本发布后，createClass 更是被废弃，当我们使用它的时候，会提示报错，也就是说，在 react 团队看来 createClass 已经完全没有存在的必要了。

其实 Class Component 完全替代 React.createClass 并不是说 React.createClass 有多坏，相反它还有一些 class Component 所没有的特性。它的废弃是由于ES6的出现，新增了 class 这一语法糖，让我们在 JavaScript 的开发中可以直接使用 extends 来扩展我们的对象，因此为了与标准的ES6接轨，原有的只在 react 中使用的 createClass 自然而然也成为了被抛弃的对象。但 class Component 在刚出现的时候也仍然存在的不小的争议，因为这两者还是存在一定的差别的，比如当时在Stack Overflow便出现了关于这两者的讨论，感兴趣的朋友可以去看看：

> https://stackoverflow.com/questions/30668464/react-component-vs-react-createclass

总的来说，除了语法上存在差异外，Class Component 和 React.createClass 的区别主要是以下两点（详情可以看看上面的回答）：
- React.createClass 会正确绑定 this，而 React.Component 则不行，我们需要在 constructor 里面使用 bind 或者直接使用箭头函数来绑定 this。
- React.Component 不能使用 React mixins 特性，这一方面我们可以使用高阶组件来弥补。
### 三、Class Component
Class Component创建的方式也很简单，就是普通的ES6的class的语法，通过extends来创建一个新的对象来创建react组件，下面是使用class Component创建一个组件的例子（由于为了给后面聊一聊hooks，所以在这里我使用了antd的例子）

```JavaScript
class Modal extends React.Component {
  state = { visible: false }

  showModal = () => {
    this.setState({
      visible: true,
    });
  }
  handleOk = (e) => {
    console.log(e);
    this.setState({
      visible: false,
    });
  }
  handleCancel = (e) => {
    console.log(e);
    this.setState({
      visible: false,
    });
  }
  render() {
    return (
      <div>
        <Button type="primary" onClick={this.showModal}>
          Open Modal
        </Button>
        <Modal
          title="Basic Modal"
          visible={this.state.visible}
          onOk={this.handleOk}
          onCancel={this.handleCancel}
        >
          <p>this is a modal</p>
        </Modal>
      </div>
    );
  }
}
```
上面就是antd中一个简单的 modal 组件的例子，其内部就是通过维护 visible 的状态来控制这个 modal 是否显示。我们可以看到，其中的一些方法都是使用箭头函数的方式来将 this 绑定到正确的属性。（具体为什么要这么做，不清楚的朋友可以看看下面这篇文章：）
> https://www.freecodecamp.org/news/this-is-why-we-need-to-bind-event-handlers-in-class-components-in-react-f7ea1a6f93eb/

而类似于上面的这种组件，也是近两年来我们在日常开发中使用最多的组件开发的方式。那为什么到了现在，我们又开始要强调使用 Function Component 来进行开发了呢？主要是由于 Class Component 所开发的组件仍然存在以下一些问题：
1. this 绑定的问题：
我们前面也提到了，我们在使用原本的 React.createClass 时并不需要去考虑this绑定的问题，而现在我们却要时刻注意使用bind或者箭头函数来让this正确绑定，同时也让一些新上手react的同学的上手成本有所提升。虽然这不是React的锅，但这方面的问题仍然客观存在。
2. 嵌套地狱： 这种情况则多发生于需要用到Context的场景下，在这种场景下，数据是同步的，因为需要通知更新所有有引用到数据的地方，因此我们就需要通过render-props 的形式定义在Context.Consumer的children中，而使用到越多的Context 就会导致嵌套层级越多，这很容易让人看代码看的一脸懵逼。比如这样：

```JavaScript
<FirstContext.Consumer>
  {first => (
    <SecondContext.Consumer>
      {second => (
        <ThirdContext.Consumer>
          {third => (
            <Component />
          )}
        </ThirdContext.Consumer>
      )}
    </SecondContext.Consumer>
  )}
</FirstContext.Consumer>
```
3. Life-cycles 的问题：生命周期函数也是我们在日常开发所经常使用到的东西。虽然生命周期函数用起来很方便，但一旦组件的逻辑变得复杂起来，这些生命周期函数也会变得难以理解和维护；同时如何让这些生命周期函数与react渲染有效结合也是一个不小的问题，这往往可能会让一些刚上手的人摸不着头脑。此外使用这些生命周期函数时也可能会出现一些预料之外的事情发生（比如在某些生命周期函数中进行数据请求，而导致组件被重复渲染多次的问题等等，这些都是有可能发生的）
> 详细可以去看看知乎上的这个回答：[https://www.zhihu.com/question/300049718](https://www.zhihu.com/question/300049718)

### 四、Function Component
看到这里，大家对class Component所存在的一些问题也算是有一些了解了，但为什么它还能横行如此之久，一直占据着主流的地位呢？其本质上就是因为没有竞争对手嘛，Function Component 长期没有内部状态管理机制，只能通过外部来管理状态，因此组件的可测试性非常的高，写起来也简洁明了，符合现在前端函数式的大潮流，是个好同志。但也正是因为没有状态管理机制，所以无法和Class Component相抗衡，毕竟一旦组件内部的逻辑变得复杂之后，内部的状态管理机制是必须的。

因此 React 团队基于 Function Component 提出 Hooks 的概念，用以解决 Function Component 的内部状态管理，同时也希望通过 Hooks 来解决 Class Component 所存在的问题。下面就是使用 Hooks 针对 antd 中的 modal 进行的改写，大家可以自行感受一下：

```JavaScript
const Modal = () => {
  const [visible , changeVisible] = useState(false)
  return (
    <div>
      <Button type="primary" onClick={()=>changeVisible(true)}>open</Button>
      <Modal
          title="Basic Modal"
          visible={visible}
          onOk={()=>changeVisible(false)}
          onCancel={()=>changeVisible(false)}
        >
          <p>this is a modal</p>
        </Modal>
    </div>
  )
}
```
我们可以看到，基于 Function Component 与 Hooks 所编写出来的组件代码是相当简洁明了的，也直接避免了我们上面所提到的 this 指向的问题。而对于上面所提到的嵌套地狱以及 Life-cycles 的问题，Hooks也提供了 useContext 和 useEffect（这个倒还是存在一些问题） 来解决，在这里我也不详细说了，详情可以去看官方文档或者是 Dan 的博客：
> https://overreacted.io/a-complete-guide-to-useeffect/

好了，看到这里我想大家都以为上面 Class Component 的问题都已经得到圆满解决了，Function Component好像已经圆满了，我们只管放心的使用它就好了。但世界上哪有这么好的事情，Function Component 仍然存在着下面几个 tip 是我们在使用前要知道的：
1. Function Component 与 Class Component 表现不同，这块不清楚的可以直接去看Dan的文章，他对这方面做了很明白的阐述：
> https://overreacted.io/how-are-function-components-different-from-classes/
2. 使用useState需要注意的是，它的执行顺序要在每次 render 时必须保持一致，不可以进判断和循环，必须写在最前面，关于这一点看视频：
> https://www.youtube.com/watch?v=dpw9EHDh2bM
3. Function Component 中，外部对与函数式组件的操作只能通过 props 来进行控制，不能通过函数式组件内部暴露方法来对组件进行操作。

参考资料：
- https://ultimatecourses.com/blog/react-create-class-versus-component
- https://overreacted.io/how-are-function-components-different-from-classes/
- https://www.youtube.com/watch?v=dpw9EHDh2bM
- https://overreacted.io/a-complete-guide-to-useeffect/
- [https://www.zhihu.com/question/300049718](https://www.zhihu.com/question/300049718)
- https://stackoverflow.com/questions/30668464/react-component-vs-react-createclass
- http://taobaofed.org/blog/2018/11/27/hooks-and-function-component/
- https://www.freecodecamp.org/news/this-is-why-we-need-to-bind-event-handlers-in-class-components-in-react-f7ea1a6f93eb/