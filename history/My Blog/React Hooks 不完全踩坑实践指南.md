
## 前言
使用Hooks开发也有一年多了，期间踩了一些坑，也收获了一些体会，撑国庆有点时间小小的总结梳理了一下，后续有时间会一直补充。

# 一、常用的 Hook 以及它们的一些注意事项

## 1.1 useState
useState 就不用多说了，它的作用就在于可以让函数式组件拥有状态。在使用它的时候，需要注意的是：**永远不要将可以通过计算得到的值保存为 state**。包括但不限于：

- 从 props 传递而来的值，但不能直接使用，需要通过一些计算来对数据进行处理，才能进行使用。常见的场景为：前端实现的搜索，在保持搜索值 searchValue 的同时，还维护一个搜索后的列表 searchedList 的 state。实际上这个 searchedList 是不必要的，我们实际上只需要维护 searchValue 即可。
- 从 URL、cookie、localStorage 中获取到的值，对于这些从前端存储中获取的，我们都应该是即用即取，而不是单独使用一个状态去将其维护起来。

这样做的好处有：

1. **保证了状态的最小化**。一般情况下，状态的数量和代码的复杂程度是成正比的。当我们维护了很多的状态时，会需要注意各个状态之间的依赖关系，以及它们对于组件渲染的影响。因此，在一般情况下，保持状态的最小化，有利于我们写出易于维护的组件。
2. **保持唯一数据源**。这一点主要对应于我们上述所说的从前端存储以及url上获取的值转化为state，从而增加中间状态。我曾自身体会过以及多次见过，前端的状态来源于 url 或者 前端存储，但没有做好状态同步，而导致这些来源值变化时，前端组件维护的状态没有更新而导致的bug。因此，正确的方式应该是，直接从数据的来源获取数据，即去即用，做到数据源以及用数据的地方的双向同步即可。

## 1.2 useEffect
useEffect，顾名思义即执行副作用的地方。很多情况下，使用过 class 组件的同学会将其与 class 组件中的一些生命周期函数相绑定。但其实它的机制是不太一样的，class 组件的生命周期函数是按照一定的标准去顺序执行的，而 useEffect 则是每次组件 render 完后判断其 **依赖 **然后执行的。

对于useEffect 的依赖，有以下几点可以参考的：

1. 单个useEffect 依赖的值别过多，一般最好别超过 5 个。因为一旦过多，就表示这个useEffect有频繁调用或者预料之外调用的风险。这时，我们可以考虑对无关状态进行剔除，以及相关状态的合并，以及对单个 useEffect 进行分离，让单个 useEffect 的职责变得更小。
2. 依赖项一般是一个常量数组，而不是一个变量，早创建回调时，就应该明确要依赖于哪些值了。
3. 依赖值定义的变量一定是会在回调函数中用到的。
4. React 是用浅比较来对依赖项进行比较的，因此对于依赖项为数组或者对象的情况下，很容易出现bug。

## 1.3 useCallback 和 useMemo
我自己在面试时，很喜欢问一些熟悉 react 技术栈的同学一个问题：在函数式组件中，是否需要对那些 inline 的函数都套上一层 useCallback 来提升性能？有不少同学回答的是：是，需要。

咋一看，因为react函数组件会在UI发生变化时重新执行一次。因此如果每次重新执行，都需要创建这些 inline 的函数，可能会对性能造成影响，因此需要用 useCallback 包上一层。但实际上，JavaScript对于创建函数式很快的（[官方解释](https://reactjs.org/docs/hooks-faq.html#are-hooks-slow-because-of-creating-functions-in-render)），相反纯粹的给一个组件套上 useCallback 只会更慢，因此 inline 函数无论如何都会创建，useCallback还需要比较依赖项的变化。

因此我们应该在以下一些场景下，才考虑使用 useCallback：

- 该函数在初始化的时候需要大量的计算（这种场景极少）
- 该函数会作为props传递给子组件供子组件使用（这种情况较多）
- 该函数会作为其他 hooks 的依赖项（这种情况也较少）

而 useMemo 也是用于缓存，和 useCallback 不同的是，useCallback 缓存的是函数引用，而useMemo则是缓存计算结果。通常用于：**一个数据是根据其他数据计算而来的，且这个数据只有当其他数据发生变化时，发需要重新计算。**

有意思的是，useCallback其实也可以通过useMemo来进行实现，因为它们本质上都是做同一件事情：**建立一个结果和数据的依赖关系，只有依赖数据发生变化，结果才会发生改变。**
```typescript
const myUseCallback = useMemo(() => {
  // 返回一个函数作为结果
	return () => {}
}, [deps])
```

## 1.4 useRef
ref 是 react 中一直存在的。在class组件中，我们通常使用 createRef 来创建ref；而在函数组件中，我们则使用 useRef 来创建 ref。这会让人直观的感觉 useRef 就是 createRef 在函数组件的替代品，但其实它们还是存在差异的。useRef 的 current 像一个变量，其.current属性被初始化为传入的参数(initialValue)，返回的对象将在组件的整个生命周期内持续存在。而createRef 则会在每次渲染都会返回一个新的引用，而 useRef 每次都会返回相同的引用。因此，当我们使用 useRef 时，通常我们是需要在**多次渲染之间共享一个数据**；除此之外，也可用于保存某个 DOM 节点的引用，对某个节点去进行操作。

至于其他需要注意的是：useRef 的内容发生变化时，它不会通知我们，且更改 current 属性也不会导致重新刷新，因为它只是一个引用而已。

# 二、自定义 Hook
自定义 hook 也是React开发中非常重要的一环。我们可以通过自定义hook来有效的提升我们代码的可复用性以及可维护性。在使用 hook 的过程中，我想大家不难发现，hook 带来的收益其实主要是在于两点：

1. 逻辑复用
2. 关注分离

而自定义hook，其实也主要是集中于这两点来进行扩展开发的，其使用场景大体上也主要是以下两种场景：

1. 封装通用逻辑
2. 对复杂组件进行拆分

## 2.1 封装通用逻辑
对通用逻辑进行封装，是自定义 hook 中非常常见的一种使用场景。譬如说，我们以往在 class 组件中经常使用 componentDidUpdate 来实现组件更新是才需要触发的一些逻辑。然而在函数组件中，我们并不能直接使用 useEffect 来实现这个功能，因为 useEffect 的执行时和 render 相关联的，因此在组件初次渲染时也会执行。在这种情况下，我们就可以封装一个 useDidUpdate 来实现类似于 componentDidUpdate 的功能：
```typescript
const useDidUpdate = (callback: () => void, array: unknown[]) => {
  const mounted = useRef(false);
  useEffect(() => {
    if (mounted.current) {
      callback();
    } else {
      mounted.current = true;
    }
  }, [...array]);
};
```
这是从通用逻辑的角度的自定义 hook 的封装，对于一些业务逻辑，我们同样也可以进行处理。譬如说在微前端方案中，主子应用的通信也是其中重要的一环，而 BPM 中单据的审批也需要主子应用进行通信，然后进行相应的回调函数的执行。考虑到这种情况，在经过一些调研后，我当时就选用了使用 [CustomEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/CustomEvent) 来作为我们主子应用的通信方案，用自定义 hook 便可对这种业务逻辑进行抽象，实现复用：
```typescript
import { useEffect } from 'react';

type Options = boolean | AddEventListenerOptions;

interface CustomEventParams<T> {
  key: string;
  callback?: (e: CustomEvent<T>) => void;
  options?: Options;
}
/**
 * @param key - 一个表示 event 名字的字符串
 * @return callback: (data) => void
 */
export const useEmitter = <T>(key: string) => {
  const callback = (data: T) => {
    const event = new CustomEvent(key, { detail: data });
    window.dispatchEvent(event);
  };

  return callback;
};

/**
 * @param key - 一个表示 event 名字的字符串
 * @param callback - 回调函数
 * @param options - 可选参数
 * @return void
 */
export const useListener = <T>(
  key: string,
  callback: (e: CustomEvent<T>) => void,
  options: Options = {}
) => {
  useEffect(() => {
    if (typeof callback === 'function') {
      const fn = (e: Event) => {
        callback(e as CustomEvent);
      };

      window.addEventListener(key, fn, options);

      return () => window.removeEventListener(key, fn, options);
    }
  }, [key, callback, options]);
};
/**
 * @param key - 一个表示 event 名字的字符串
 * @param callback - 回调函数
 * @param options - 可选参数
 * @return useEmitter:（key: string）=> callback
 */
export const useCustomEvent = <T>({ key, callback, options = {} }: CustomEventParams<T>) => {
  let fn;
  if (typeof callback === 'function') {
    fn = callback;
  } else {
    fn = () => console.log('no function');
  }
  useListener<T>(key, fn, options);

  return useEmitter<T>(key);
};

```
大家看到以上主要有三个API组成：

- useEmitter
- useListener
- useCustomEvent

其中，在BPM的业务场景中， useEmitter 单独使用的较少，而  useListener 则通常适用于监听业务方传递过来的相关消息，而 useCustomEvent 则主要使用于在 BPM 发出事件后，根据业务方的回传消息作出响应的场景。

## 2.2 对复杂组件进行拆分
我相信很多同学在接手一些老项目的代码时，时常会发现有些组件的代码量会超出自己的控制范围（譬如我接手的一个项目，一个组件写了7000行，里面嵌套的一个组件也有6000行）。这样的代码是非常不好维护的，因此保持每个组件的短小，是一个非常有用的最佳实践。

那如何做到不让单个组件变得太过于冗余呢？其实做法很简单，就是尽量将相关的逻辑做成独立的 hooks，然后在函数组件中使用这些 Hooks。其实在 Vue 中也有类似的做法， Vue 的文档对于这种代码组织的方式介绍的很清楚，在这里我也就不做过多的赘述了：[why-composition-api ](https://v3.vuejs.org/guide/composition-api-introduction.html#why-composition-api )。其核心要点就在于，**对单个大的组件，进行业务逻辑上的分离，将拥有共同逻辑的代码拆分为 hooks，从而降低代码的耦合程度，减少单个组件的代码量。**


# 三、其他

## 3.1 使用 ESlint 插件监督 Hooks 的使用
由于 Hooks 是通过闭包和数组组成的环形链表来实现的，因此在开发过程中，我们也需要遵循一些开发规范才行，包括：

- 在不能在条件语句、循环、return之后使用 hooks。
- hooks只能在函数组件以及自定义 hooks 中使用
- useEffect的回调函数使用的变量，都必须在依赖项中进行声明

这些规范虽然我们都知道，但难免有时会忘记，因此我们可以通过 [eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks) 来检查我们 hooks的使用情况，在下载完这个插件后，直接在配置文件加上这两个配置即可：
```typescript
    "react-hooks/exhaustive-deps": "warn",
    "react-hooks/rules-of-hooks": "error",
```

## 3.2 自定义Hook注释
对于自定义的 hooks，在定义完后尽量加上较为完备的注释，注释的方式也推荐使用 ts doc 的格式去进行声明，这样一方面有利于后来的开发者（也可能是你自己）可以较快的明白这个自定义hook 所要完成的目的，另一方面也可能让你自己在写这个 hooks 的时候，可以二次思考这个 hooks 的入参和返回值是否合理。譬如说，上述的 useCustomEvent：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1633781378395-322dca49-ef98-4a6c-ad42-75852dd368ab.png#clientId=uee6368bf-bbf4-4&from=paste&height=349&id=u72866a38&name=image.png&originHeight=1046&originWidth=2518&originalType=binary&ratio=1&size=267673&status=done&style=none&taskId=u776a85b4-64c1-4e3a-9eff-23691f9278d&width=839.3333333333334)<br />我们只需看注释即可只要这个hooks的入参以及返回值。

参考资料：

- [https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)
- [https://time.geekbang.org/column/intro/100079901](https://time.geekbang.org/column/intro/100079901)
- [https://v3.vuejs.org/guide/composition-api-introduction.html#why-composition-api](https://v3.vuejs.org/guide/composition-api-introduction.html#why-composition-api)
- [https://reactjs.org/docs/hooks-faq.html#are-hooks-slow-because-of-creating-functions-in-render](https://reactjs.org/docs/hooks-faq.html#are-hooks-slow-because-of-creating-functions-in-render)
