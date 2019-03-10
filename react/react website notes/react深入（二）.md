## 非受控组件

在大多数情况下，我们推荐使用 [受控组件](https://react.docschina.org/docs/forms.html) 来实现表单。 在受控组件中，表单数据由 React 组件处理。如果让表单数据由 DOM 处理时，替代方案为使用非受控组件。

在非受控组件中接收单个属性：

```
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
  }
  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.value);
    event.preventDefault();
  }
  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={(input) => this.input = input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

如果依然不清楚在哪种特定情况下选择哪种类型的组件，那么你应该阅读 [这篇关于受控和非受控的表单输入](http://goshakkk.name/controlled-vs-uncontrolled-inputs-react/) 了解更多。

#### 默认值

在 React 的生命周期中，表单元素上的 `value` 属性将会覆盖 DOM 中的值。使用非受控组件时，通常你希望 React 可以为其指定初始值，但不再控制后续更新。要解决这个问题，你可以指定一个 `defaultValue` 属性而不是 `value`。

```
render() {
  return (
    <form onSubmit={this.handleSubmit}>
      <label>
        Name:
        <input
          defaultValue="Bob"
          type="text"
          ref={(input) => this.input = input} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}
```

同样，`<input type="checkbox">` 和 `<input type="radio">` 支持 `defaultChecked`，`<select>` 和 `<textarea>` 支持 `defaultValue`.

#### 文件输入标签

在HTML中，`<input type="file">` 可以让用户从其设备存储中选择一个或多个文件上传到服务器，或通过[File API](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications)进行操作。

```
<input type="file" />
```

在React中，`<input type="file" />` 始终是一个不受控制的组件，因为它的值只能由用户设置，而不是以编程方式设置

## 性能优化

#### 使用生产版本

在React应用中检测性能问题时，请务必使用压缩过的生产版本。

默认情况下，React包含很多在开发过程中很有帮助的警告。然而，这会导致React更大更慢。因此，在部署应用时，请确认使用了生产版本。

如果你不确定构建过程是否正确，可以安装[React开发者工具（chrome）](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)。当你访问一个生产模式的React页面时，这个工具的图标会有一个黑色的背景：

![1544755571537](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1544755571537.png)

当你访问一个开发模式的React页面时，这个工具的图标会有一个红色的背景：

![1544755599800](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1544755599800.png)

最好在开发应用时使用开发模式，部署应用时换为生产模式。

#### Create React App方式

```
npm run build
```

**注意只有发布项目时才有必要这样做，正常开发时，使用`npm start`。**

#### 单文件生产版本

我们提供压缩好的生产版本的React和React DOM文件:

```
<script src="https://unpkg.com/react@15/dist/react.min.js"></script>
<script src="https://unpkg.com/react-dom@15/dist/react-dom.min.js"></script>
```

注意只有结尾为`.min.js`的React文件才是适合生产使用的

#### Browserify

为了创建最高效的Browserify生产版本，需要安装一些插件：

```
# If you use npm
npm install --save-dev bundle-collapser envify uglify-js uglifyify 

# If you use Yarn
yarn add --dev bundle-collapser envify uglify-js uglifyify 
```

为了构建生产版本，务必添加这些设置指令 **(参数很重要)**:

- [`envify`](https://github.com/hughsk/envify)该插件确保正确的编译环境，全局安装（`-g`）。
- [`uglifyify`](https://github.com/hughsk/uglifyify)该插件移除了开发接口。全局安装（`-g`）。
- [`bundle-collapser`](https://github.com/substack/bundle-collapser)该插件用数字替代了长长的模块ID。
- 最后，以上结果都被输添加至[`uglify-js`](https://github.com/mishoo/UglifyJS2)来得到整合。([了解原因](https://github.com/hughsk/uglifyify#motivationusage)).

#### Rollup

为了创建最高效的Rollup生产版本，需要安装一些插件：

```
# If you use npm
npm install --save-dev rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-uglify 

# If you use Yarn
yarn add --dev rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-uglify 
```

为了构建生产版本，务必添加这些插件 **(参数很重要)**:

- [`replace`](https://github.com/rollup/rollup-plugin-replace)该插件确保正确的编译环境。
- [`commonjs`](https://github.com/rollup/rollup-plugin-commonjs)该插件在Rollup内提供对CommonJS的支持。
- [`uglify`](https://github.com/TrySound/rollup-plugin-uglify)该插件压缩生成最终版本。

```
plugins: [
  // ...
  require('rollup-plugin-replace')({
    'process.env.NODE_ENV': JSON.stringify('production')
  }),
  require('rollup-plugin-commonjs')(),
  require('rollup-plugin-uglify')(),
  // ...
]
```

查看完整的[安装例子](https://gist.github.com/Rich-Harris/cb14f4bc0670c47d00d191565be36bf0).

注意只有生产版本需要这样操作。不要在开发环境中安装`uglify`和`replace`，因为它们会隐藏掉有用的React警告并使构建过程更慢。

#### Webpack

> **注意：**
>
> 如果你正在使用[Create React App](https://react.docschina.org/docs/optimizing-performance.html#create-react-app)方式，参考上述文档。
> 本节只适用于直接配置Webpack的情况。

为了创建最高效的Webpack生产版本，需要在生产版本的配置中添加这些插件：

```
new webpack.DefinePlugin({
  'process.env': {
    NODE_ENV: JSON.stringify('production')
  }
}),
new webpack.optimize.UglifyJsPlugin()
```

了解更多参见[Webpack文档](https://webpack.js.org/guides/production-build/).

注意只有生产版本需要这样操作。不要在开发环境中安装`UglifyJsPlugin`和`DefinePlugin`，因为它们会隐藏掉有用的React警告并使构建过程更慢。

#### shouldComponentUpdate应用

这是一个组件的子树。对其中每个组件来说，`SCU`表明了`shouldComponentUpdate`的返回内容，`vDOMEq`表明了待渲染的React元素与原始元素是否相等，最后，圆圈的颜色表明这个组件是否需要重新渲染。

![1544769555548](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1544769555548.png)

由于以C2为根的子树的`shouldComponentUpdate`返回了`false`，React不会试图渲染C2，甚至不会在C4和C5上调用`shouldComponentUpdate`。

对C1和C3来说，`shouldComponentUpdate`返回了`true`，因此React会深入到分支中并检查它们。C6的`shouldComponentUpdate`返回了`true`，由于待渲染的元素与原始元素并不相等，React会更新这个DOM节点。

最后一个有趣的情况是C8，React需要渲染这个组件，但是由于组件元素返回值与原元素相等，因此它并没有更新这个DOM节点。

注意React只需更新C6，因为它是不可避免的。对C8来说，它通过比较待渲染元素与原始元素避免了渲染，对C2的子树和C7，它们甚至都没有执行比较，因为我们设置了`shouldComponentUpdate`为`false`，`render`没有被调用。

#### 使用不可突变的数据结构

[Immutable.js](https://github.com/facebook/immutable-js)是解决这个问题的另一种方法。它通过结构共享提供不可突变的，持久的集合：

- *不可突变*:一旦创建，集合就不能在另一个时间点改变。
- *持久性*:可以使用原始集合和一个突变来创建新的集合。原始集合在新集合创建后仍然可用。
- *结构共享*:新集合尽可能多的使用原始集合的结构来创建，以便将复制操作降至最少从而提升性能

```
const SomeRecord = Immutable.Record({ foo: null });
const x = new SomeRecord({ foo: 'bar' });
const y = x.set('foo', 'baz');
x === y; // false
```

在这个例子中，`x`突变后返回了一个新的索引，因此我们可以安全的确认`x`被改变了。

还有两个库可以帮助我们使用不可突变数据：[seamless-immutable](https://github.com/rtfeldman/seamless-immutable) 和[immutability-helper](https://github.com/kolodny/immutability-helper)。

实现`shouldComponentUpdate`时，不可突变的数据结构帮助我们轻松的追踪对象变化。这通常可以提供一个不错的性能提升。

### Keys

实践中，发现key通常不难。你将展示的元素可能已经带有一个唯一的ID，因此key可以来自于你的数据中：

```
<li key={item.id}>{item.name}</li>
```

当这已不再是问题，你可以给你的数据增加一个新的ID属性，或根据数据的某些内容创建一个哈希值来作为key。key必须在其兄弟节点中是唯一的，而非全局唯一。

万不得已，你可以传递他们在数组中的索引作为key。若元素没有重排，该方法效果不错，但重排会使得其变慢。

当索引用作key时，组件状态在重新排序时也会有问题。组件实例基于key进行更新和重用。如果key是索引，则item的顺序变化会改变key值。这将导致非受控组件的状态可能会以意想不到的方式混淆和更新。

[这里](https://reactjs.org/redirect-to-codepen/reconciliation/index-used-as-key)是在CodePen上使用索引作为键可能导致的问题的一个例子，[这里](https://reactjs.org/redirect-to-codepen/reconciliation/no-index-used-as-key)是同一个例子的更新版本，展示了如何不使用索引作为键将解决这些reordering, sorting, 和 prepending的问题。

## Context

Context 通过组件树提供了一个传递数据的方法，从而避免了在每一个层级手动的传递 props 属性。

### 何时使用 Context

Context 设计目的是为共享那些被认为对于一个组件树而言是“全局”的数据，例如当前认证的用户、主题或首选语言。

```
// 创建一个 theme Context,  默认 theme 的值为 light
const ThemeContext = React.createContext('light');
function ThemedButton(props) {
  // ThemedButton 组件从 context 接收 theme
  return (
    <ThemeContext.Consumer>
      {theme => <Button {...props} theme={theme} />}
    </ThemeContext.Consumer>
  );
}
// 中间组件
function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}
class App extends React.Component {
  render() {
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}
```

**不要仅仅为了避免在几个层级下的组件传递 props 而使用 context，它是被用于在多个层级的多个组件需要访问相同数据的情景。**

### API

#### `React.createContext`

```
const {Provider, Consumer} = React.createContext(defaultValue);
```

创建一对 `{ Provider, Consumer }`。当 React 渲染 context 组件 Consumer 时，它将从组件树的上层中最接近的匹配的 Provider 读取当前的 context 值。

如果上层的组件树没有一个匹配的 Provider，而此时你需要渲染一个 Consumer 组件，那么你可以用到 `defaultValue` 。这有助于在不封装它们的情况下对组件进行测试。

#### `Provider`

```
<Provider value={/* some value */}>
```

React 组件允许 Consumers 订阅 context 的改变。

接收一个 `value` 属性传递给 Provider 的后代 Consumers。一个 Provider 可以联系到多个 Consumers。Providers 可以被嵌套以覆盖组件树内更深层次的值。

#### `Consumer`

```
<Consumer>
  {value => /* render something based on the context value */}
</Consumer>
```

一个可以订阅 context 变化的 React 组件。

接收一个 [函数作为子节点](https://react.docschina.org/docs/render-props.html#using-props-other-than-render). 函数接收当前 context 的值并返回一个 React 节点。传递给函数的 `value` 将等于组件树中上层 context 的最近的 Provider 的 `value` 属性。如果 context 没有 Provider ，那么 `value` 参数将等于被传递给 `createContext()` 的 `defaultValue` 。

> 注意
>
> 关于此案例的更多信息, 请看 [render props](https://react.docschina.org/docs/render-props.html).

每当Provider的值发生改变时, 作为Provider后代的所有Consumers都会重新渲染。 从Provider到其后代的Consumers传播不受shouldComponentUpdate方法的约束，因此即使祖先组件退出更新时，后代Consumer也会被更新。

通过使用与[Object.is](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description)相同的算法比较新值和旧值来确定变化。

> 注意
>
> （这在传递对象作为 `value` 时会引发一些问题[Caveats](https://react.docschina.org/docs/context.html#caveats).）

### 作用于多个上下文

为了保持 context 快速进行二次渲染， React 需要使每一个 Consumer 在组件树中成为一个单独的节点。

```
// 主题上下文, 默认light
const ThemeContext = React.createContext('light');
// 登陆用户上下文
const UserContext = React.createContext();
// 一个依赖于两个上下文的中间组件
function Toolbar(props) {
  return (
    <ThemeContext.Consumer>
      {theme => (
        <UserContext.Consumer>
          {user => (
            <ProfilePage user={user} theme={theme} />
          )}
        </UserContext.Consumer>
      )}
    </ThemeContext.Consumer>
  );
}
class App extends React.Component {
  render() {
    const {signedInUser, theme} = this.props;

    // App组件提供上下文的初始值
    return (
      <ThemeContext.Provider value={theme}>
        <UserContext.Provider value={signedInUser}>
          <Toolbar />
        </UserContext.Provider>
      </ThemeContext.Provider>
    );
  }
}
```

如果两个或者多个上下文的值经常被一起使用，也许你需要考虑你自己渲染属性的组件提供给它们。