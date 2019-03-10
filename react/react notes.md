## react复习点

**为什么虚拟dom会提高性能?**

虚拟dom相当于在js和真实dom中间加了一个缓存，利用dom diff算法避免了没有必要的dom操作，从而提高性能。

具体实现步骤如下：

用 JavaScript 对象结构表示 DOM 树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中

当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异

把2所记录的差异应用到步骤1所构建的真正的DOM树上，视图就更新了。

**diff算法?**

把树形结构按照层级分解，只比较同级元素。

给列表结构的每个单元添加唯一的key属性，方便比较。

React 只会匹配相同 class 的 component（这里面的class指的是组件的名字）

合并操作，调用 component 的 setState 方法的时候, React 将其标记为 dirty.到每一个事件循环结束, React 检查所有标记 dirty 的 component 重新绘制.

选择性子树渲染。开发人员可以重写shouldComponentUpdate提高diff的性能。

**react性能优化方案**

（1）重写shouldComponentUpdate来避免不必要的dom操作。

（2）使用 production 版本的react.js。

（3）使用key来帮助React识别列表中所有子组件的最小变化。

**setState是一个异步的过程，它会集齐一批需要更新的组件然后一起更新**。

#### React 中 setState 什么时候是同步的，什么时候是异步的？

在 React 中，如果是由 React 引发的事件处理（比如通过 onClick 引发的事件处理），调用 setState 不会同步更新 this.state，除此之外的 setState 调用会同步执行 this.state。所谓“除此之外”，指的是绕过 React 通过 addEventListener 直接添加的事件处理函数，还有通过 setTimeout/setInterval 产生的异步调用。

**原因：**在 React 的 setState 函数实现中，会根据一个变量 **isBatchingUpdates** 判断是直接更新 this.state 还是放到队列中回头再说，而 isBatchingUpdates 默认是 false，也就表示 setState 会同步更新 this.state，但是，有一个函数 batchedUpdates，这个函数会把 isBatchingUpdates 修改为t rue，而当 React 在调用事件处理函数之前就会调用这个 batchedUpdates，造成的后果就是由 React 控制的事件处理过程 setState 不会同步更新 this.state。

#### React setState 笔试题，下面的代码输出什么？

```
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
复制代码
```

解析：

1、第一次和第二次都是在 react 自身生命周期内，触发时 isBatchingUpdates 为 true，所以并不会直接执行更新 state，而是加入了 dirtyComponents，所以打印时获取的都是更新前的状态 0。

2、两次 setState 时，获取到 this.state.val 都是 0，所以执行时都是将 0 设置成 1，在 react 内部会被合并掉，只执行一次。设置完成后 state.val 值为 1。

3、setTimeout 中的代码，触发时 isBatchingUpdates 为 false，所以能够直接进行更新，所以连着输出 2，3。

输出： 0 0 2 3

