## 高阶组件

高阶组件（HOC）是react中对组件逻辑进行重用的高级技术。但高阶组件本身并不是React API。它只是一种模式，这种模式是由react自身的组合性质必然产生的。

具体而言，**高阶组件就是一个函数，且该函数接受一个组件作为参数，并返回一个新的组件**

```
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

对比组件将props属性转变成UI，高阶组件则是将一个组件转换成另一个新组件。

高阶组件在React第三方库中很常见，比如Redux的[`connect`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options)方法和Relay的[`createContainer`](https://facebook.github.io/relay/docs/api-reference-relay.html#createcontainer-static-method).

### 使用高阶组件（HOC）解决交叉问题

从 `DataSource` 订阅数据并调用 `setState` 的模式将会一次又一次的发生。我们就可以抽象出一个模式，该模式允许我们在一个地方定义逻辑并且能对所有的组件使用，这就是高阶组件的精华所在。

```
// 函数接受一个组件参数……
function withSubscription(WrappedComponent, selectData) {
  // ……返回另一个新组件……
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }
    componentDidMount() {
      // ……注意订阅数据……
      DataSource.addChangeListener(this.handleChange);
    }
    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }
    render() {
      // ……使用最新的数据渲染组件
      // 注意此处将已有的props属性传递给原组件
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

注意，高阶组件既不会修改input原组件，也不会使用继承复制input原组件的行为。相反，高阶组件是通过将原组件 *包裹（wrapping）* 在容器组件（container component）里面的方式来 *组合（composes）* 使用原组件。高阶组件就是一个没有副作用的纯函数。

就是这样！包裹组件接收容器组件的所有props属性以及一个新的 `data`属性，并用 `data` 属性渲染输出内容。高阶组件并不关心数据是如何以及为什么被使用，而包裹组件也不关心数据来自何处。

因为 `withSubscription` 就是一个普通函数，你可以添加任意数量的参数。例如，你或许会想使 `data` 属性可配置化，使高阶组件和包裹组件进一步隔离开。或者你想要接收一个参数用于配置 `shouldComponentUpdate` 函数，或配置数据源的参数。这些都可以实现，因为高阶组件可以完全控制新组件的定义。

和普通组件一样，`withSubscription` 和包裹组件之间的关联是完全基于 props 属性的。这就使为组件切换一个 HOC 变得非常轻松，只要保证备选的几种高阶组件向包裹组件提供是相同类型的 props 属性即可。

### **不要改变原始组件，使用组合**

不要在高阶组件内部修改（或以其它方式修改）原组件的原型属性。

高阶组件（修改原型的高阶组件）对没有生命周期函数的无状态函数式组件也是无效的。

更改型高阶组件（mutating HOCs）泄露了组件的抽象性 —— 使用者必须知道他们的具体实现，才能避免与其它高阶组件的冲突。

高阶组件应该使用组合技术。

```
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log('Current props: ', this.props);
      console.log('Next props: ', nextProps);
    }
    render() {
      // 用容器组件组合包裹组件且不修改包裹组件，这才是正确的打开方式。
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

组合型高阶组件对类组件和无状态函数式组件适用性同样好。而且，因为它是一个纯函数，它和其它高阶组件，甚至它自身也是可组合的。

**容器组件是专注于在高层次和低层次关注点之间进行责任划分的策略的一部分。容器组件会处理诸如数据订阅和状态管理等事情，并传递props属性给展示组件。而展示组件则负责处理渲染UI等事情。高阶组件使用容器组件作为实现的一部分。你也可以认为高阶组件就是参数化的容器组件定义。**

### 约定：将不相关的props属性传递给包裹组件

```
render() {
  // 过滤掉与高阶函数功能相关的props属性，
  // 不再传递
  const { extraProp, ...passThroughProps } = this.props;
  // 向包裹组件注入props属性，一般都是高阶组件的state状态
  // 或实例方法
  const injectedProp = someStateOrInstanceMethod;
  // 向包裹组件传递props属性
  return (
    <WrappedComponent
      injectedProp={injectedProp}
      {...passThroughProps}
    />
  );
}
```

### 约定：最大化使用组合

大部分常见高阶组件的函数签名如下所示：

```
// React Redux's `connect`
const ConnectedComment = connect(commentSelector, commentActions)(Comment);
```

```
// 不要这样做……
const EnhancedComponent = withRouter(connect(commentSelector)(WrappedComponent))

// ……你可以使用一个功能组合工具
// compose(f, g, h) 和 (...args) => f(g(h(...args)))是一样的
const enhance = compose(
  // 这些都是单参数的高阶组件
  withRouter,
  connect(commentSelector)
)
const EnhancedComponent = enhance(WrappedComponent)
```

`connect`函数产生的高阶组件和其它增强型高阶组件具有同样的被用作装饰器的能力。

### 约定：包装显示名字以便于调试

高阶组件创建的容器组件在[`React Developer Tools`](https://github.com/facebook/react-devtools)中的表现和其它的普通组件是一样的。为了便于调试，可以选择一个好的名字，确保能够识别出它是由高阶组件创建的新组件还是普通的组件。

最常用的技术就是将包裹组件的名字包装在显示名字中。所以，如果你的高阶组件名字是 `withSubscription`，且包裹组件的显示名字是 `CommentList`，那么就是用 `WithSubscription(CommentList)`这样的显示名字：

```
function withSubscription(WrappedComponent) {
  class WithSubscription extends React.Component {/* ... */}
  WithSubscription.displayName = `WithSubscription(${getDisplayName(WrappedComponent)})`;
  return WithSubscription;
}
function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}
```

### 注意事项

**1.不要在render函数中使用高阶组件**

```
render() {
  // 每一次render函数调用都会创建一个新的EnhancedComponent实例
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // 每一次都会使子对象树完全被卸载或移除
  return <EnhancedComponent />;
}
```

这里产生的问题不仅仅是性能问题 —— 还有，重新加载一个组件会引起原有组件的所有状态和子组件丢失。

相反，在组件定义外使用高阶组件，可以使新组件只出现一次定义。在渲染的整个过程中，保证都是同一个组件。无论在任何情况下，这都是最好的使用方式。

在很少的情况下，你可能需要动态的调用高阶组件。那么你就可以在组件的构造函数或生命周期函数中调用。

**2.必须将静态方法做拷贝**

当使用高阶组件包装组件，原始组件被容器组件包裹，也就意味着新组件会丢失原始组件的所有静态方法。

```
// 定义静态方法
WrappedComponent.staticMethod = function() {/*...*/}
// 使用高阶组件
const EnhancedComponent = enhance(WrappedComponent);

// 增强型组件没有静态方法
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

解决这个问题的方法就是，将原始组件的所有静态方法全部拷贝给新组件：

```
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // 必须得知道要拷贝的方法 :(
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```

这样做，就需要你清楚的知道都有哪些静态方法需要拷贝。你可以使用[hoist-non-react-statics](https://github.com/mridgway/hoist-non-react-statics)来帮你自动处理，它会自动拷贝所有非React的静态方法：

```
import hoistNonReactStatic from 'hoist-non-react-statics';
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
```

另外一个可能的解决方案就是分别导出组件自身的静态方法。

```
// 替代……
MyComponent.someFunction = someFunction;
export default MyComponent;

// ……分别导出……
export { someFunction };

// ……在要使用的组件中导入
import MyComponent, { someFunction } from './MyComponent.js';
```

**3.Refs属性不能传递**

一般来说，高阶组件可以传递所有的props属性给包裹的组件，但是不能传递refs引用。因为并不是像`key`一样，refs是一个伪属性，React对它进行了特殊处理。如果你向一个由高阶组件创建的组件的元素添加ref应用，那么ref指向的是最外层容器组件实例的，而不是包裹组件。

如果你碰到了这样的问题，最理想的处理方案就是搞清楚如何避免使用 `ref`。有时候，没有看过React示例的新用户在某种场景下使用prop属性要好过使用ref。

现在我们提供一个名为 `React.forwardRef` 的 API 来解决这一问题（在 React 16.3 版本中）。[在 refs 传递章节中了解详情](https://react.docschina.org/docs/forwarding-refs.html)。

## Render Props

术语 [“render prop”](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) 是指一种在 React 组件之间使用一个值为函数的 prop 在 React 组件间共享代码的简单技术。

带有 render prop 的组件带有一个返回一个 React 元素的函数并调用该函数而不是实现自己的渲染逻辑。

```
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```

使用 render props 的库包括 [React Router](https://reacttraining.com/react-router/web/api/Route/Route-render-methods) 和 [Downshift](https://github.com/paypal/downshift)。

### 在交叉关注点（Cross-Cutting Concerns）使用 render props

```
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}
class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }
  render() {
    return (
      <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>

        {/*
          Instead of providing a static representation of what <Mouse> renders,
          use the `render` prop to dynamically determine what to render.
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}
class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```

提供了一个 `render` prop 以让 `<Mouse>` 能够动态决定什么需要渲染，而不是克隆 `<Mouse>` 组件并硬编码来解决特定的用例。

更具体地说，**render prop 是一个组件用来了解要渲染什么内容的函数 prop。**

关于 render props 一个有趣的事情是你可以使用一个带有 render props 的常规组件来实现大量的 [高阶组件](https://react.docschina.org/docs/higher-order-components.html) (HOC)。例如，如果你更偏向于使用一个 `withMouse` 的高阶组件而不是一个 `<Mouse>` 组件，你可以轻松的创建一个带有 render prop 的常规 `<Mouse>` 组件的高阶组件。

```
// If you really want a HOC for some reason, you can easily
// create one using a regular component with a render prop!
function withMouse(Component) {
  return class extends React.Component {
    render() {
      return (
        <Mouse render={mouse => (
          <Component {...this.props} mouse={mouse} />
        )}/>
      );
    }
  }
}
```

所以，使用 render props 是一种可能的使用模式

### 使用 Props 而非 `render`

记住仅仅是因为这一模式被称为 “render props” 而你*不必为使用该模式而用一个名为 render 的 prop*。实际上，[组件能够知道什么需要渲染的*任何*函数 prop 在技术上都是 “render prop” ](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce)。

尽管之前的例子使用了 `render`，我们也可以简单地使用 `children` prop！

```
<Mouse children={mouse => (
  <p>The mouse position is {mouse.x}, {mouse.y}</p>
)}/>
```

并记住，`children` prop 并不真正需要添加到 JSX 元素的 “attributes” 列表中。相反，你可以直接放置到元素的*内部*！

```
<Mouse>
  {mouse => (
    <p>The mouse position is {mouse.x}, {mouse.y}</p>
  )}
</Mouse>
```

你可以在 [react-motion](https://github.com/chenglou/react-motion) API 里看到这一技术的使用。

由于这一技术有些不寻常，当你在设计一个类似的 API 时，你可能要直接地在你的 `propTypes`里声明 `children` 应为一个函数类型。

```
Mouse.propTypes = {
  children: PropTypes.func.isRequired
};
```

### 在 React.PureComponent 中使用 render props 要注意

如果你在 `render` 方法里创建函数，那么使用 render prop 会抵消使用 [`React.PureComponent`](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce)带来的优势。这是因为浅 prop 比较对于新 props 总会返回 `false`，并且在这种情况下每一个 `render` 对于 render prop 将会生成一个新的值。

```
class Mouse extends React.PureComponent {
  // Same implementation as above...
}
class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>

        {/*
          This is bad! The value of the `render` prop will
          be different on each render.
        */}
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```

在这样例子中，每次 `<MouseTracker>` 渲染，它会生成一个新的函数作为 `<Mouse render>` 的 prop，因而在同时也抵消了继承自 `React.PureComponent` 的 `<Mouse>` 组件的效果！

定义一个 prop 作为实例方法，类似如下：

```
class MouseTracker extends React.Component {
  constructor(props) {
    super(props);

    // This binding ensures that `this.renderTheCat` always refers
    // to the *same* function when we use it in render.
    this.renderTheCat = this.renderTheCat.bind(this);
  }

  renderTheCat(mouse) {
    return <Cat mouse={mouse} />;
  }

  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={this.renderTheCat} />
      </div>
    );
  }
}
```

`<Mouse>` 应继承`React.Component`，万一你没法提前在构造函数中绑定实例方法（如因为你可能要掩盖组件的 props 和/或 state)。

## 严格模式

`StrictMode`是一个用以标记出应用中潜在问题的工具。就像`Fragment`，`StrictMode`不会渲染任何真实的UI。它为其后代元素触发额外的检查和警告。

> 注意: 严格模式检查只在开发模式下运行，不会与生产模式冲突。
>
> ```
> import React from 'react';
> function ExampleApplication() {
>   return (
>     <div>
>       <Header />
>       <React.StrictMode>
>         <div>
>           <ComponentOne />
>           <ComponentTwo />
>         </div>
>       </React.StrictMode>
>       <Footer />
>     </div>
>   );
> }
> ```

`StrictMode`目前有助于：

- [识别具有不安全生命周期的组件](https://react.docschina.org/docs/strict-mode.html#%E8%AF%86%E5%88%AB%E5%85%B7%E6%9C%89%E4%B8%8D%E5%AE%89%E5%85%A8%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E7%9A%84%E7%BB%84%E4%BB%B6)
- [有关旧式字符串ref用法的警告](https://react.docschina.org/docs/strict-mode.html#%E6%9C%89%E5%85%B3%E6%97%A7%E5%BC%8F%E5%AD%97%E7%AC%A6%E4%B8%B2ref%E7%94%A8%E6%B3%95%E7%9A%84%E8%AD%A6%E5%91%8A)
- [检测意外的副作用](https://react.docschina.org/docs/strict-mode.html#%E6%A3%80%E6%B5%8B%E6%84%8F%E5%A4%96%E7%9A%84%E5%89%AF%E4%BD%9C%E7%94%A8)

### 识别具有不安全生命周期的组件

如同在[博客](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html)中阐明的，在异步React应用中使用某些老式的生命周期方法不安全。但是, 如果应用程序使用第三方库, 则很难确保不使用这些生命周期方法。幸运的是, 严格的模式可以帮助解决这个问题!

当启用严格模式, React将编译一个所有使用不安全生命周期组件的列表，并打印一条关于这些组件的警告信息，就像：

![1545095789998](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1545095789998.png)

### 有关旧式字符串ref用法的警告

以前，React提供了2种方法管理ref：旧式的字符串ref API和回调API。虽然字符串ref API更加方便，但它有些许[缺点](https://github.com/facebook/react/issues/1373)，因此我们的正式建议是[改用回调方式](https://doc.react-china.org/docs/refs-and-the-dom.html#%E6%97%A7%E7%89%88-api%EF%BC%9Astring-%E7%B1%BB%E5%9E%8B%E7%9A%84-refs)

```
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.inputRef = React.createRef();
  }
  render() {
    return <input type="text" ref={this.inputRef} />;
  }
  componentDidMount() {
    this.inputRef.current.focus();
  }
}
```

由于新增的对象式refs很大程度上作为字符串ref的替换，因此strict mode现在对字符串ref的用法发出警告。

> 注意： 除了新的createRef API，回调ref将被继续支持。您不需要在组件中替换回调ref。它们稍微灵活一些, 因此它们将保持为高级功能。

### 检测意外的副作用

理论上，React在两个阶段起作用:

- **渲染**阶段决定了需要对 DOM 进行哪些更改。在此阶段, React调用`render`(方法), 然后将结果与上一次渲染进行比较。
- **提交**阶段是React执行任何更改的阶段。(在React DOM中, 指React插入、更新和删除 dom 节点）。在此阶段React也调用生命周期, 如 `componentDidMount` 和 `componentDidUpdate` 。

提交阶段通常很快，但是渲染可能很慢。因此, 即将出现的异步模式 (默认情况下尚未启用) 将呈现工作分解为片断, 暂停和恢复工作以避免阻止浏览器。这意味着在提交之前, 反应可能不止一次地调用渲染阶段生命周期, 或者它可以在不提交的情况下调用它们 (因为错误或更高的优先级中断)。

渲染阶段的生命周期包括以下class component方法：

- `constructor`
- `componentWillMount`
- `componentWillReceiveProps`
- `componentWillUpdate`
- `getDerivedStateFromProps`
- `shouldComponentUpdate`
- `render`
- `setState` 更新函数 (第一个形参）

因为以上方法可能不止一次被调用，所以它们中不包含副作用尤为重要。忽略此规则可能会导致各种问题, 包括内存泄漏和无效的应用程序状态。不幸的是, 很难发现这些问题, 因为它们通常都是[不确定的](https://en.wikipedia.org/wiki/Deterministic_algorithm)。

严格模式不能自动检测到你的副作用, 但它可以帮助你发现它们, 使其更具确定性。这是通过有意地双调用以下方法来完成的:

- Class component `constructor`
- `render`
- `setState` 更新函数 (第一个形参）
- static `getDerivedStateFromProps`

> 注意： 只在开发模式生效。生产模式下生命周期不会被双调用。

