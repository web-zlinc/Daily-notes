## 一、UI 组件

React-Redux 将所有组件分成两大类：UI 组件（presentational component）和容器组件（container component）。

UI 组件有以下几个特征。

> - 只负责 UI 的呈现，不带有任何业务逻辑
> - 没有状态（即不使用`this.state`这个变量）
> - 所有数据都由参数（`this.props`）提供
> - 不使用任何 Redux 的 API

下面就是一个 UI 组件的例子。

> ```
> const Title =
>   value => <h1>{value}</h1>;
> ```

因为不含有状态，UI 组件又称为"纯组件"，即它纯函数一样，纯粹由参数决定它的值。

## 二、容器组件

容器组件的特征恰恰相反。

> - 负责管理数据和业务逻辑，不负责 UI 的呈现
> - 带有内部状态
> - 使用 Redux 的 API

总之，只要记住一句话就可以了：**UI 组件负责 UI 的呈现，容器组件负责管理数据和逻辑**。

你可能会问，如果一个组件既有 UI 又有业务逻辑，那怎么办？回答是，将它拆分成下面的结构：外面是一个容器组件，里面包了一个UI 组件。前者负责与外部的通信，将数据传给后者，由后者渲染出视图。

React-Redux 规定，所有的 UI 组件都由用户提供，容器组件则是由 React-Redux 自动生成。也就是说，用户负责视觉层，状态管理则是全部交给它。

## 三、connect()

React-Redux 提供`connect`方法，**用于从 UI 组件生成容器组件。`connect`的意思，就是将这两种组件连起来**。

> ```
> import { connect } from 'react-redux'
> const VisibleTodoList = connect()(TodoList);
> ```

上面代码中，`TodoList`是 UI 组件，`VisibleTodoList`就是由 React-Redux 通过`connect`方法自动生成的容器组件。

但是，因为没有定义业务逻辑，上面这个容器组件毫无意义，只是 UI 组件的一个单纯的包装层。为了定义业务逻辑，需要给出下面两方面的信息。

> （1）输入逻辑：外部的数据（即`state`对象）如何转换为 UI 组件的参数
>
> （2）输出逻辑：用户发出的动作如何变为 Action 对象，从 UI 组件传出去。

因此，`connect`方法的完整 API 如下。

> ```
> import { connect } from 'react-redux'
> 
> const VisibleTodoList = connect(
>   mapStateToProps,
>   mapDispatchToProps
> )(TodoList)
> ```

上面代码中，`connect`方法接受两个参数：`mapStateToProps`和`mapDispatchToProps`。它们定义了 UI 组件的业务逻辑。前者负责输入逻辑，即将`state`映射到 UI 组件的参数（`props`），后者负责输出逻辑，即将用户对 UI 组件的操作映射成 Action。

### 四、mapStateToProps()

- `mapStateToProps`是一个函数。它的作用就是像它的名字那样，建立一个从（外部的）`state`对象到（UI 组件的）`props`对象的映射关系。

- 作为函数，`mapStateToProps`执行后应该返回一个对象，里面的每一个键值对就是一个映射。请看下面的例子。


> ```
> const mapStateToProps = (state) => {
>   return {
>     todos: getVisibleTodos(state.todos, state.visibilityFilter)
>   }
> }
> ```

上面代码中，`mapStateToProps`是一个函数，它接受`state`作为参数，返回一个对象。这个对象有一个`todos`属性，代表 UI 组件的同名参数，后面的`getVisibleTodos`也是一个函数，可以从`state`算出 `todos` 的值。

下面就是`getVisibleTodos`的一个例子，用来算出`todos`。

> ```
> const getVisibleTodos = (todos, filter) => {
>   switch (filter) {
>     case 'SHOW_ALL':
>       return todos
>     case 'SHOW_COMPLETED':
>       return todos.filter(t => t.completed)
>     case 'SHOW_ACTIVE':
>       return todos.filter(t => !t.completed)
>     default:
>       throw new Error('Unknown filter: ' + filter)
>   }
> }
> ```

`mapStateToProps`会订阅 Store，每当`state`更新的时候，就会自动执行，重新计算 UI 组件的参数，从而触发 UI 组件的重新渲染。

`mapStateToProps`的第一个参数总是`state`对象，还可以使用第二个参数，代表容器组件的`props`对象。

> ```
> // 容器组件的代码
> //    <FilterLink filter="SHOW_ALL">
> //      All
> //    </FilterLink>
> 
> const mapStateToProps = (state, ownProps) => {
>   return {
>     active: ownProps.filter === state.visibilityFilter
>   }
> }
> ```

使用`ownProps`作为参数后，如果容器组件的参数发生变化，也会引发 UI 组件重新渲染。

`connect`方法可以省略`mapStateToProps`参数，那样的话，UI 组件就不会订阅Store，就是说 Store 的更新不会引起 UI 组件的更新。

，这个 UI 组件有两个参数：`value`和`onIncreaseClick`。前者需要从`state`计算得到，后者需要向外发出 Action。

接着，定义`value`到`state`的映射，以及`onIncreaseClick`到`dispatch`的映射。

> ```
> function mapStateToProps(state) {
>   return {
>     value: state.count
>   }
> }
> 
> function mapDispatchToProps(dispatch) {
>   return {
>     onIncreaseClick: () => dispatch(increaseAction)
>   }
> }
> 
> // Action Creator
> const increaseAction = { type: 'increase' }
> ```

然后，使用`connect`方法生成容器组件。

> ```
> const App = connect(
>   mapStateToProps,
>   mapDispatchToProps
> )(Counter)
> ```

然后，定义这个组件的 Reducer。

> ```
> // Reducer
> function counter(state = { count: 0 }, action) {
>   const count = state.count
>   switch (action.type) {
>     case 'increase':
>       return { count: count + 1 }
>     default:
>       return state
>   }
> }
> ```

最后，生成`store`对象，并使用`Provider`在根组件外面包一层。

> ```
> import { loadState, saveState } from './localStorage';
> 
> const persistedState = loadState();
> const store = createStore(
>   todoApp,
>   persistedState
> );
> 
> store.subscribe(throttle(() => {
>   saveState({
>     todos: store.getState().todos,
>   })
> }, 1000))
> 
> ReactDOM.render(
>   <Provider store={store}>
>     <App />
>   </Provider>,
>   document.getElementById('root')
> );
> ```

完整的代码看[这里](https://github.com/jackielii/simplest-redux-example/blob/master/index.js)。

### 五、React-Router 路由库

使用`React-Router`的项目，与其他项目没有不同之处，也是使用`Provider`在`Router`外面包一层，毕竟`Provider`的唯一功能就是传入`store`对象。

> ```
> const Root = ({ store }) => (
>   <Provider store={store}>
>     <Router>
>       <Route path="/" component={App} />
>     </Router>
>   </Provider>
> );
> ```

 