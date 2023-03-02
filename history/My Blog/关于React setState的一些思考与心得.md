
### 前言

这篇文章主要是为了纪录一些自己对于setState的认识的不断深入的过程。我觉得这过程对我自己来说很有价值，不光是层层递进的了解一个API的执行机制，更对react总体的设计有了更为深入的认识。



### 第一阶段 初识setState

使用过React的应该都知道，在React中，一个组件中要读取当前状态需要访问this.state，但是更新状态却需要使用this.setState，不是直接在this.state上修改，就比如这样：

```javascript
//读取状态
const count = this.state.count；

//更新状态
this.setState({count: count + 1})；

//无意义的修改
this.state.count = count + 1;
```

其实这主要有几点考虑，首先this.state说到底只是一个对象，单纯的去修改一个对象的值是毫无意义的，在React中只有去驱动UI的更新才有意义，因此虽然我们可以尝试直接改变this.state，但并没有驱动UI的重新渲染，因此这种操作也就毫无意义。也正是由于这个原因，我们就需要使用this.setState来驱动组件的更新过程。

然后在我刚学习React时，我就看见了这段很经典的代码：

```javascript
function incrementMultiple() {
  this.setState({count: this.state.count + 1});
  this.setState({count: this.state.count + 1});
  this.setState({count: this.state.count + 1});
}
```

作为一名JSer，我看完就毫不犹豫的想到，这不就是count的值加3么。但转眼看了下面的答案，光速打脸，实际的结果是state只增加了1。然后我就不由想到当时没怎么看懂的React文档中的一些话：状态更新可能是异步的，状态更新合并。恩，没毛病，因为异步且会合并，因此这三条语句合并为一条语句了，所以就只执行一次。然后就扭头溜了，并没有去思考一些深层次的问题。



### 第二阶段 setState理解的进阶

但是随着对React的理解的逐步加深，我开始对setState有了更加深的理解：

首先我意识到this.setState会通过引发一次组件的更新过程来引发重新绘制。也就是说setState的调用会引起React的更新生命周期的四个函数的依次调用：

- shouldComponentUpdate
- componentWillUpdate
- render
- componentDidUpdate

我们都知道，在React生命周期函数里，以render函数为界，无论是挂载过程和更新过程，在render之前的几个生命周期函数，this.state和Props都是不会发生更新的，直到render函数执行完毕后，this.state才会得到更新。（有一个例外：当shouldComponentUpdate函数返回false，这时候更新过程就被中断了，render函数也不会被调用了，这时候React不会放弃掉对this.state的更新的，所以虽然不调用render，依然会更新this.state。）

React的官方文档有提到过这么一句话：

> 状态更新会合并（也就是说多次setstate函数调用产生的效果会合并）。


起初我对这句话理解并不是很深刻,但按照官方文档的代码示例写了这么一段代码：

```javascript
function updateName() {
  this.setState({Age: '22'})
  this.setState({Name: 'srtian'})
}
```

果然执行结果与以下代码是等价的

```javascript
function updateName() {
  this.setState({Age: '22', Name: 'srtian})
}
```

于是我将其理解为一个队列，每个this.setState()都会被合并起来，排成一排，到最后一次解决。但对其设计的原因并不理解，只知道这样有利于性能（也是在文档上看到的）。

直到理解上面React生命周期函数的原理后，我才理解了setState关于这个设计的意图。

前面我们提到过，每一次使用setState都会调用一次更新的生命周期，如果每一次this.setState()都调用一次上面那四个生命周期函数，虽然以上四个函数都是纯函数，性能浪费上还好说，但render函数会将结果拿去做Virtual DOM比较和更新DOM树，这个就比较费时间。因此，将多个this.setSate进行合并，render函数就能够将合并后的this.setState()的结果一次性的与Virtual DOM比较然后更新DOM树，这样就能够用有效的提升性能。

除此之外，我还认为setState的设计十分巧妙，一般来说只在render函数后才会进行更新this.state。这其实也避免了React16的Fiber可能会产生的一个问题：由于Fiber下的组件更新是可以中断，也就是说在一个组件的更新过程中，可能更新到一半的时候就由于其他原因而中断更新，回去做更重要的事情了，在做完更重要的事情后，再回来更新这个组件，这会导致前面的那些生命周期函数可能会执行多次。因此如果在render之前this.setState()就改变状态的话，很有可能就会导致组件状态的多次更新，从而导致组件状态的混乱。



### 第三阶段 从源码理解setstate

> 这是React15.6版本，由于React16变动较大，setState的调用栈发生变动，因此仅供参考。


经历了上面那个阶段，我算是对setState有那么一些理解了，但还是不能理解很多东西比如：this.setState()的是怎么合并的？setState()到底是怎样一种骚操作？...等等。然后我又看见了这段经典的代码：

```javascript
class Example extends React.Component {
  constructor() {
    super();
    this.state = {
      val: 0
    };
  }
  
  componentDidMount() {
    this.setState({val: this.state.val + 1});
    console.log(this.state.val);    // 第 1 次 log

    this.setState({val: this.state.val + 1});
    console.log(this.state.val);    // 第 2 次 log

    setTimeout(() => {
      this.setState({val: this.state.val + 1});
      console.log(this.state.val);  // 第 3 次 log

      this.setState({val: this.state.val + 1});
      console.log(this.state.val);  // 第 4 次 log
    }, 0);
  }

  render() {
    return null;
  }
};
```

恩！按照我多年经验，这波操作我看不懂！

![](https://tse1-mm.cn.bing.net/th?id=OIP.ZlvZC7ugr-G8oLHt_9NXTAHaDv&w=200&h=98&c=8&rs=1&qlt=90&dpr=1.25&pid=3.1&rm=2#align=left&display=inline&height=122&originHeight=122&originWidth=250&status=done&width=250)

于是硬着头皮打开了React源码，开始一波瞎分析：<br />
首先就是setState了，可以看出它接受两个参数 partialState 和callback，其中 partialState 顾名思义就是部分 state，起这个名字也能就是想表达它的 state 没有改变（瞎猜的。。。）。以下是省略了一部分的代码，只看核心部分。

```javascript
ReactComponent.prototype.setState = function(partialState, callback) {
  invariant(
    typeof partialState === 'object' ||
      typeof partialState === 'function' ||
      partialState == null,
    'setState(...): takes an object of state variables to update or a ' +
      'function which returns an object of state variables.',
  );
  this.updater.enqueueSetState(this, partialState);
  if (callback) {
    this.updater.enqueueCallback(this, callback, 'setState');
  }
};
 enqueueSetState: function(publicInstance, partialState) {
    if (__DEV__) {
      ReactInstrumentation.debugTool.onSetState();
      warning(
        partialState != null,
        'setState(...): You passed an undefined or null state object; ' +
          'instead, use forceUpdate().',
      );
    }

    var internalInstance = getInternalInstanceReadyForUpdate(
      publicInstance,
      'setState',
    );

    if (!internalInstance) {
      return;
    }

    var queue =
      internalInstance._pendingStateQueue ||
      (internalInstance._pendingStateQueue = []);
    queue.push(partialState);

    enqueueUpdate(internalInstance);
  }
// 通过enqueueUpdate执行state更新
function enqueueUpdate(component) {
  ensureInjected();
  // batchingStrategy是批量更新策略，isBatchingUpdates表示是否处于批量更新过程
  // 最开始默认值为false
  if (!batchingStrategy.isBatchingUpdates) {
    batchingStrategy.batchedUpdates(enqueueUpdate, component);
    return;
  }
  dirtyComponents.push(component);

  if (component._updateBatchNumber == null) {
    component._updateBatchNumber = updateBatchNumber + 1;
  }
}
// 对_pendingElement, _pendingStateQueue, _pendingForceUpdate进行判断，
// _pendingStateQueue由于会对state进行修改，所以不为空，
// 然后会调用updateComponent方法
performUpdateIfNecessary: function(transaction) {
    if (this._pendingElement != null) {
      ReactReconciler.receiveComponent(
        this,
        this._pendingElement,
        transaction,
        this._context,
      );
    } else if (this._pendingStateQueue !== null || this._pendingForceUpdate) {
      this.updateComponent(
        transaction,
        this._currentElement,
        this._currentElement,
        this._context,
        this._context,
      );
    } else {
      this._updateBatchNumber = null;
    }
  },
```

其中这段代码需要额外注意：

```javascript
// batchingStrategy是批量更新策略，isBatchingUpdates表示是否处于批量更新过程
  // 最开始默认值为false
if (!batchingStrategy.isBatchingUpdates) {
    batchingStrategy.batchedUpdates(enqueueUpdate, component);
    return;
  }
dirtyComponents.push(component);
if (component._updateBatchNumber == null) {
    component._updateBatchNumber = updateBatchNumber + 1;
  }
```

上面这段代码的意思就是如果是处于批量更新模式，也就是isBatchingUpdates为true时，不进行state的更新操作，而是将需要更新的component添加到dirtyComponents数组中；如果不处于批量更新模式，对所有队列中的更新执行batchedUpdates方法，然后往下看下去就知道是用事务的方式批量的进行component的更新。

然后可以找到了这个batchedUpdates：

```javascript
var ReactDefaultBatchingStrategy = {
  // 也就是上面提到的默认为false
  isBatchingUpdates: false,
  // 这是一个方法，只有在isBatchingUpdates: false时才会调用
  // 但一般来说，出于react大事务中时会在render中的_renderNewRootComponent中将变为true。
  batchedUpdates: function(callback, a, b, c, d, e) {
    var alreadyBatchingUpdates = ReactDefaultBatchingStrategy.isBatchingUpdates;
    ReactDefaultBatchingStrategy.isBatchingUpdates = true;
    // The code is written this way to avoid extra allocations
    if (alreadyBatchingUpdates) {
      return callback(a, b, c, d, e);
    } else {
      return transaction.perform(callback, null, a, b, c, d, e);
    }
  },
```

看到这我总算理解了，当我们调用setState时，最终会通过enqueueUpdate执行state更新，就像上面那样有两种更新的模式，一种是批量更新模式，将组建保存在dirtyComponents;另一种非批量模式，将会遍历dirtyComponents，对每一个dirtyComponents调用updateComponent方法。就像这张图：

![](https://gitee.com/uploads/images/2018/0528/145734_c2bb7d17_1575229.jpeg#align=left&display=inline&height=490&originHeight=490&originWidth=720&status=done&width=720)

至于批量与非批量模式，会通过ReactDefaultBatchingStrategy中的isBatchingUpdates属性来进行判断。在非批量模式下，会立即应用新的state；而在批量模式下，需要更新state的组件会被push 到dirtyComponents，再执行更新。

所以我们再看前面的那坨代码：

```javascript
class Example extends React.Component {
  constructor() {
    super();
    this.state = {
      val: 0
    };
  }
  
  componentDidMount() {
    this.setState({val: this.state.val + 1});
    console.log(this.state.val);    // 第 1 次 log

    this.setState({val: this.state.val + 1});
    console.log(this.state.val);    // 第 2 次 log

    setTimeout(() => {
      this.setState({val: this.state.val + 1});
      console.log(this.state.val);  // 第 3 次 log

      this.setState({val: this.state.val + 1});
      console.log(this.state.val);  // 第 4 次 log
    }, 0);
  }

  render() {
    return null;
  }
};
```

就不难看出它的答案是 0, 0, 2, 3。

总结起来就是这样：

- this.setState首先会把state推入pendingState队列中
- 然后将组件标记为dirty
- React中有事务的概念，最常见的就是更新事务，如果不在事务中，则会开启一次新的更新事务，更新事务执行的操作就是把组件标记为dirty。
- 判断是否处于batch update
- 是的话，保存组建于dirtyComponent中，在事务结束的时候才会通过 ReactUpdates.flushBatchedUpdates 方法将所有的临时 state merge 并计算出最新的 props 及 state，然后将其批量执行，最后再关闭结束事务。
- 不是的话，直接开启一次新的更新事务，在标记为dirty之后，直接开始更新组件。因此当setState执行完毕后，组件就更新完毕了，所以会造成定时器同步更新的情况。

另外还有就是updateComponent方法，这也很重要：

```javascript
{   // 会检测组件中的state和props是否发生变化，有变化才会进行更新; 
    // 如果shouldUpdateComponent函数中返回false则不会执行组件的更新
    updateComponent: function (transaction,
                               prevParentElement,
                               nextParentElement,
                               prevUnmaskedContext,
                               nextUnmaskedContext,) {
        var inst = this._instance;
        var nextState = this._processPendingState(nextProps, nextContext);
        var shouldUpdate = true;

        if (!this._pendingForceUpdate) {
            if (inst.shouldComponentUpdate) {
                if (__DEV__) {
                    shouldUpdate = measureLifeCyclePerf(
                            () => inst.shouldComponentUpdate(nextProps, nextState, nextContext),
                            this._debugID,
                            'shouldComponentUpdate',
                    );
                } else {
                    shouldUpdate = inst.shouldComponentUpdate(
                            nextProps,
                            nextState,
                            nextContext,
                    );
                }
            } else {
                if (this._compositeType === CompositeTypes.PureClass) {
                    shouldUpdate =
                            !shallowEqual(prevProps, nextProps) ||
                            !shallowEqual(inst.state, nextState);
                }
            }
        }
        
    },
// 该方法会合并需要更新的state，然后加入到更新队列中
    _processPendingState: function (props, context) {
        var inst = this._instance;
        var queue = this._pendingStateQueue;  
        var replace = this._pendingReplaceState;
        this._pendingReplaceState = false; 
        this._pendingStateQueue = null;

        if (!queue) {
            return inst.state;
        }

        if (replace && queue.length === 1) {
            return queue[0];
        }

        var nextState = Object.assign({}, replace ? queue[0] : inst.state);
        for (var i = replace ? 1 : 0; i < queue.length; i++) {
            var partial = queue[i];
            Object.assign(
                    nextState,
                    typeof partial === 'function'
                            ? partial.call(inst, nextState, props, context)
                            : partial,
            );
        }

        return nextState;
    }
};
```

发现它会调用shouldComponentUpdate和componentWillUpdate方法，看到这不由理解了一个定律：不要在shouldComponentUpdate和componentWillUpdate中调用setState。如果在这两个生命周期里调用setState，会造成循环调用。
