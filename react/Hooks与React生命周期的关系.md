React 生命周期很多人都了解，但通常我们所了解的都是 单个组件 的生命周期，但针对 Hooks 组件、多个关联组件（父子组件和兄弟组件） 的生命周期又是怎么样的喃？你有思考和了解过吗，接下来我们将完整的了解 React 生命周期。

## Hooks 组件

`函数组件`的本质是函数，没有 state 的概念的，因此不存在生命周期一说，仅仅是一个 render 函数而已。

但是引入`Hooks`
之后就变得不同了，它能让组件在不使用 class 的情况下拥有 state，所以就有了生命周期的概念

即：Hooks 组件（使用了 Hooks 的函数组件）有生命周期，而函数组件（未使用 Hooks 的函数组件）是没有生命周期的

下面，是具体的 class 与 Hooks 的生命周期对应关系

- constructor
  函数组件不需要构造函数，我们可以通过调用`useState`来初始化 state
- getDerivedStateFromProps
  一般情况下，我们不需要使用它，我们可以在`渲染过程中更新state`，以达到实现 getDerivedStateFromProps 的目的
- shouldComponentUpdate
  可以用`React.memo`包裹一个组件对它的 props 进行浅比较
  `const Comp = React.memeo((props) => { // 具体的组件 })`
  注意： React.memo 等价于 PureComponent，它只浅比较 props，这里也可以使用 useMemo 优化每一个节点
- render
  这是函数组件体本身
- componentDidMount
  可以使用`useEffect`处理副作用
  `useEffect(() => { // 在componentDidMount执行的内容 }, [count]) // 仅在count更改时更新`
- componentWillUnmount
  相当于 useEffect 里面返回的 cleanup 函数
  `useEffect(() => { // 需要在componentDidMount执行的内容 return function cleanup() { // 需要在componentWillUnmount执行的内容 } })`
- componentDidCatch && getDerivedStateFromError
  目前还没有这些方法的 Hook 等价写法

大致汇总成表格如下：

| class 组件               | Hooks 组件                |
| :----------------------- | :------------------------ |
| constructor              | useState                  |
| getDerivedStateFromProps | useState 里面 update 函数 |
| shouldComponentUpdate    | useMemo                   |
| render                   | 函数本身                  |
| componentDidMount        | useEffect                 |
| componentDidUpdate       | useEffect                 |
| componentWillUnmount     | useEffect 里面返回的函数  |
| componentDidCatch        | 无                        |
| getDerivedStateFromError | 无                        |

## 单个组件的生命周期

组件的生命周期可以分为三个阶段:

- 挂载阶段
- 组件更新阶段
- 挂载阶段

### v16.3 之前

- 挂载阶段 - constructor - componentWillMount - render: react 最重要的步骤，创建虚拟 dom，进行 diff 算法，更新 dom 树都在此进行 - componentDidMount
- 组件更新阶段 - componentWillReceiveProps - shouldComponentUpdate - componentWillUpdate - render - componentDidUpdate
- 卸载阶段 - componentWillUnmount

这种生命周期会存在一个问题，那就是当更新复杂组件的最上层组件时，调用栈会很长，如果再进行复杂的操作，就可能长时间阻塞主线程，带来不好的用户体验。

所以 v16.3 引入了新的 API 来解决这个问题

### v16.3 之后

- `static getDerivedStateFromProps`: 该函数在挂载阶段和组件更新阶段都会执行，即每次获取新的 props 或 state 之后都会被执行，在挂载阶段用来代替 componentWillMount；在组件更新阶段配合 componentDidUpdate，可以覆盖 componentWillReceiveProps 的所有用法。

同时它是一个静态函数，所以函数体内不能访问 this，会根据 nextProps 和 prevState 计算出预期的状态改变，返回结果会被送给 setState，返回 null 则说明不需要更新 state，并且这个返回是必须的。

- `getSnapshotBeforeUpdate`: 该函数会在 render 之后，DOM 更新前被调用，用于读取最新的 DOM 数据。

返回一个值，作为 componentDidUpdate 的第三个参数。配合 componentDidUpdate，可以覆盖 componentWillUpdate 的所有用法。

即更新后的生命周期为：

- 挂载阶段 - constructor - static getDerivedStateFromProps - render - componentDidMount
- 更新阶段 - static getDerivedStateFromProps - shouldComponentUpdate - render - getSnapshotBeforeUpdate - componentDidUpdate
- 卸载阶段 - componentWillUnmount