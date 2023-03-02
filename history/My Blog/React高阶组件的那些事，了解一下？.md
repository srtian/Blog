
#### 前言

学习react已经有一段时间了，期间在阅读官方文档的基础上也看了不少文章，但感觉对很多东西的理解还是不够深刻，因此这段时间又在撸一个基于react全家桶的聊天App（现在还在瞎78写的阶段，在往这个聊天App这个方向写），通过实践倒是对react相关技术栈有了更为深刻的理解。在使用react-redux的过程中，发现connect好像还挺有意思的，也真实感受到了高阶组件所带来的便利，出于自己写项目本身就是为了学习的目的，因此对高阶组件又进行了一番学习。写下这篇文章主要是对高阶组件的知识进行一个梳理与总结，如有错误疏漏之处，敬请指出，不胜感激。



## 一、初识高阶组件

_要学习高阶组件首先我们要知道的就是高阶组件是什么，解决了什么样的问题。_

React官方文档的对高阶组件的说明是这样的:

> A higher-order component (HOC) is an advanced technique in React for reusing component logic. HOCs are not part of the React API, perse. They are a pattern that emerges from React’s compositional nature.


从上面的说明我们可以看出，react 的高阶组件并不是 react API 的一部分，它源自于 react 的生态。

简单来说，一个高阶组件就是一个函数，它接受一个组件作为输入，然后会返回一个新的组件作为结果，且所返回的新组件会进行相对增强。值得注意的是，我们在这说的组件并不是组件实例，而是一个组件类或者一个无状态组件的函数。就像这样：

```javascript
import React from 'react'

function removeUserProp(WrappedComponent) {
//WrappingComponent这个组件名字并不重要，它至少一个局部变量，继承自React.Component
    return class WrappingComponent extends React.Component {
        render() {
// ES6的语法，可以将一个对象的特定字段过滤掉
            const {user, ...otherProps} = this.props
            return <WrappedComponent {...otherProps} />
        }
      }
}
```

了解设计模式的大佬们应该发现了，它其实就是的设计模式中的装饰者模式在react中的应用，它通过组合的方式从而达到很高的灵活程度和复用。<br />
就像上面的代码，我们定义了一个叫做 removeUserProp 的高阶组件，传入一个叫做 WrappedComponent 的参数（代表一个组件类），然后返回一个新的组件 ，新的组件与原组件并没有太大的区别，只是将原组件中的 prop 值 user 给剔除了出来。

有了上面这个高阶组件的，当我们不希望某个组件接收到 user 时，我们就可以将这个组件作为参数传入 removeUserProp() 函数中，然后使用这个返回的组件就行了：

```javascript
const NewComponent = removeUserProp(OldComponent)
```

这样 NewComponent 组件与 OldComponent 组件拥有完全一样的行为，唯一的区别就在于传入的name属性对这个组件没有任何作用，它会自动屏蔽这个属性。也就是说，我们这个高阶组件成功的为传入的组件增加了一个屏蔽某个prop的功能。

那么明白了什么是高阶组件后，我们接下来要做的是，弄清楚高阶组件主要解决的问题，或者说我们为什么需要高阶组件？总结起来主要是以下两个方面：

1. 代码重用

> 在很多情况下，react组件都需要共用同一个逻辑，我们在这个时候就可以把这部分共用的逻辑提取出来，然后利用高阶组件的形式将其组合，从而减少很多重复的组件代码。


2.修改React组件的行为

> 很多时候有些现成的react组件并不是我们自己撸出来的，而是来自于GitHub上的大佬们的开源贡献，而当我们要对这些组件进行复用的时候，我们往往都不想去触碰这些组件的内部逻辑，这时我们就能通过高阶组件产生新的组件满足自身需求，同时也对原组件没有任何损害。


现在我们对高阶组件有了一个较为直观的认识，知道了什么是高阶组件以及高阶组件的主要用途。接下来我们就要具体了解高阶组件的实现方式以及它的具体用途了。



### 高阶组件的实现分类

对于高阶组件的实现方式我们可以根据作为参数传入的组件与返回的新组件的关系将高阶组件的实现方式分为以下两大类：

- 代理方式的高阶组件
- 继承方式的高阶组件



## 二、代理方式的高阶组件

从高阶组件的使用频率来讲，我们使用的绝大多数的高阶组件都是代理方式的高阶组件，如react-redux中的connect，还有我们在上面所实现的那个removeUserProp。这类高阶组件的特点是返回的新组件类直接继承于 React.Component 类。新组建在其中扮演的角色是一个传入参数组件的代理，在新组建的render函数中，把被包裹的组件渲染出来。在此过程中，除了高阶组件自己需要做的工作，其他的工作都会交给被包裹的组件去完成。

代理方式的高阶组件具体而言，应用场景可以分为以下几个：

- 操作prop
- 通过ref获取组件实例
- 抽取状态
- 包装组件



### 控制prop

代理类型的高阶组件返回的新组件时，渲染过程也会被新组建的render函数所控制，而在此过程中，render函数相对于一个代理，完全决定该如何使用被包裹在其中的组件。在render函数中，this.props包含了新组件接受到的所有prop。因此最直观的用法就是接受到props，然后进行任何读取，增减，修改等控制props的自定义操作。<br />
就比如我们上面的那个示例，就做到了删除prop的功能，当然我们也能实现一个添加prop的高阶组件：

```javascript
function addNewProp(WrappedComponent, newProps) {
    return class WrappingComponent extends React.Component {
        render() {
          return <WrappedComponent {...thisProps} {...newProps} />
        }
      }
}
```

这个addNewProp高阶组件与我们最开始举例的removeUserProp高阶组件在实现上并无太大的区别。唯一区别较大的就是我们传入的参数除了WrappedComponent组件类外，还新增了newProps参数。这样的高阶组件在复用性方面会跟友好，我们可以利用这样一个高阶组件给不同的组件添加不同的新属性，比如这样：

```javascript
const FirstComponent = addNewProp(OldComponent,{num: First})
const LastComponent = addNewProp(NewComponent,{num: Last})
```

在上面的代码中，我们实现了让两个完全不同的组件分别通过高阶组件生成了两个完成不同的新的组件，而这其中唯一相同的是都添加了一个属性值，且这个属性还不相同。从上面的代码我们也不难发现，高阶组件可以重用在不同组件上，减少了重复的代码。当需要注意的是，在修改和删除 Props的时候，除非由特殊的要求，否则最好不要影响到原本传递给普通组件的 Props。



### 通过ref获取组件实例

我们可以通过ref获取组件实例，但值得注意的是，React官方不提倡访问ref，我们只是讨论一下这个技术的可行性。在此我们写一个refsHOC的高阶组件，可以获得被包裹组件的ref，从而根据ref直接操纵被包裹组件的实例：

```javascript
import React from 'react'

function refsHOC(WrappedComponent) => {
  return class HOCComponent extends React.Component {
    constructor() {
      super(...arguments)
      this.linkRef = this.linkRef.bind(this)
    }
    linkRef(wrappedInstance) {
      this._root = wrappedInstance
    }
    render() {
      const props = {...this.props, ref: this.linkRef}
      return <WrappedComponent {...props}/>
    }
  }
}

export default refsHOC
```

这个refs高阶组件的工作原理其实也是增加传递给被包裹组件的props，不同的是利用了ref这个特殊的prop而已。我们通过linkRef来给被包裹组件传递ref值，linkRef被调用时，我们就可以得到被包裹组件的DOM实例。

这种高阶组件在用途上来讲可以说是无所不能的，因为只要能够获得对被包裹组件的引用，就能通过这个引用任意操纵一个组件的DOM元素，贼酸爽。但它从某个角度来讲也是啥也干不了的，因为react团队表示：**不要过度使用 Refs**。且我们也有更好的替代品——控制组件（Controlled Component)来解决相关问题，因此这个坑建议大家还是尽量少踩为好。



### 抽取状态

对于抽取状态，我想大家应该都不会很陌生。react-redux中的connect函数就实现了这种功能，它异常的强大，也成功吸引了我对高阶组件的注意。但在这有一点需要明确的是：connect函数本身并不是高阶组件，connect函数执行的结果才是一个高阶组件。让我们来看看connect的源码的主要逻辑：

```javascript
export default function connect(mapStateToProps, mapDispatchToProps, mergeProps, options = {}) {
    return function wrapWithConnect(WrappedComponent) {
        class Connect extends Component {
            constructor(props, context) {
                //参数获取
                super(props, context)
                this.store = props.store || context.store
                const storeState = this.store.getState()
                this.state = { storeState }
            }
            // 进行判断，当数据发生改变时，Component重新渲染
            shouldComponentUpdate(nextProps, nextState) {
                if (propsChanged || mapStateProducedChange || dispatchPropsChanged) {
                 this.updateState(nextProps)
                  return true
                 }
                }
            // 改变Component中的state
            componentDidMount() {
                 this.store.subscribe(() = {
                  this.setState({
                   storeState: this.store.getState()
                  })
                 })
                }
            render(){
                this.renderedElement = createElement(WrappedComponent,
                    this.mergedProps
                )
                return this.renderedElement
            }
        }
        return hoistStatics(Connect, WrappedComponent)
    }
}
```

从上面的代码我们不难看出connect模块的返回值wrapWithConnect是一个函数，而这个函数才是我们所认知的高阶组件。wrapWithConnect函数会返回一个ReactComponent对象Connect，Connect会重新render外部传入的原组件WrappedComponent，并把connect中所传入的mapStateToProps, mapDispatchToProps和this.props合并后结合成一个对象，通过属性的方式传给WrappedComponent，这才是最终的渲染结果。



### 包装组件

在日常开发中我们所接触到的大多数的高阶组件都是通过修改props部分来对输入的组件进行相对增强的。但其实高阶组件还有其他的方式来增强组件，比如我们可以通过在render函数中的JSX引入其他元素，甚至将多个react组件合并起来，来获得更骚气的样式或方法，例如我们可以给组件增加style来改变组件样式：

```javascript
const styleHOC = (WrappedComponent, style) => {
    return class HOCComponent extends React.Component {
        render() {
            return (
            <div style={style}>
                <WrappedComponent {...this.props} />
            </div>
            )
        }
    }
}
```

当我们想改变组件的样式的时候，我们就可以直接调用这个函数，比如这样：

```javascript
const style = {
			background-color: #f1fafa;
			font-family: "微软雅黑";
			font-size: 20px;
		}
const BeautifulComponent = styleHOC(uglyComponent, style)
```



## 三、继承方式的高阶组件

前面我们讨论了代理方式实现的高阶组件以及它们的主要使用方式，现在我们继续来讨论一下以继承方式实现的高阶组件。

。继承方式的高阶组件通过继承来关联作为参数传入的组件和返回的组件，比如传入的组件参数是OldComponent,那函数所返回的组件就直接继承于OldComponemt。

码界有句老话说的好：组合优于继承。在高阶组件里也不例外。<br />
继承方式的高阶组件相对于代理方式的高阶组件有很多不足之处，比如输入的组件与输出的组件共有一个生命周期等，因此通常我们接触到的高阶组件大多是代理方式实现的高阶组件，也推荐大家首先考虑以代理方式来实现高阶组件。但我们还是需要去了解并学习它，毕竟它也是有可取之处的，比如在操作生命周期函数上它还是具有其优越性的。



### 操作生命周期函数

说继承方式的高阶组件在操纵生命周期函数上有其优越性其实不够说明它在这个领域的地位，更准确地表达是：操作生命周期函数是继承方式的高阶组件所特有的功能。这是由于继承方式的高阶组件返回的新组件继承于作为参数传入的组件，两个组件的生命周期是共用的，因此可以重新定义组件的生命周期函数并作用于新组件。而代理方式的高阶组件作为参数输入的组件与输出的组件完全是两个生命周期，因此改变生命周期函数也就无从说起了。

例如我们可以定义一个让参数组件只有在用户登录时才显示的高阶组件：

```javascript
const shouldLoggedInHOC = (WrappedComponent) => {
    return class MyComponent extends WrappedComponent {
        render() {
            if (this.props.loggedIn) {
                return super.render()
            }
            else {
                return null
            }
        }
    }
}
```



### 操纵Prop

除了操作生命周期函数外，继承方式的高阶函数也能对Prop进行操作，但总的难说贼麻烦，当然也有简单的方式，比如这样：

```javascript
function removeProps(WrappedComponent) {
    return class NewComponent extends WrappedComponent {
        render() {
        const{ user, ...otherProps } = this.props
        this.props = otherProps
        return super.render()
        }
    }
}
```

虽然这样看起来很简单，但我们直接修改了this.props，这不是一个好的实践，可能会产生不可预料的后果，更好的操作办法是这样的：

```javascript
function removeProps(WrappedComponent) {
    return class NewComponent extends WrappedComponent {
        render() {
        const element =super.render()
        const{ user, ...otherProps } = this.props
        this.props = otherProps
        return React.cloneElement(element, this.props, element.props.children)
        }
    }
}
```

我们可以通过React.cloneElement来传入新的props，让这些产生的组件重新渲染一次。但虽然这种方式可以解决直接修改this.props所带来的问题，但实现起来贼麻烦，唯一用得上的就是高阶组件需要根据参数组件WrappedComponent渲染结果来决定如何修改props时用得上，其他的时候显然使用代理模式更便捷清晰。



## 四、高阶组件命名

用 HOC 包裹了一个组件会使它失去原本 WrappedComponent 的名字，可能会影响开发和debug。

因此我们通常会用 WrappedComponent 的名字加上一些 前缀作为 HOC 的名字。我们来看看React-Redux是怎么做的：

```javascript
function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName ||
         WrappedComponent.name ||
         ‘Component’
}

HOC.displayName = `HOC(${getDisplayName(WrappedComponent)})`
//或
class HOC extends ... {
  static displayName = `HOC(${getDisplayName(WrappedComponent)})`
  ...
}
```

实际上我们不用自己来写getDisplayName这个函数，recompose 提供了这个函数，我们只要使用即可。



#### 结尾语

我们其他要注意的就是[官方文档](https://react.bootcss.com/react/docs/higher-order-components.html)所说的几个约定与相关规范，在此我就不一一赘述了，感兴趣的可以自己去看看。最后很感谢能看到这里的朋友，因为水平有限，如果有错误敬请指正，十分感激！
