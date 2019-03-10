## js复习点

### 有以下 3 个判断数组的方法，请分别介绍它们之间的区别和优劣

> Object.prototype.toString.call() 、 instanceof 以及 Array.isArray()

解析：

#### 1. Object.prototype.toString.call()

每一个继承 Object 的对象都有 `toString` 方法，如果 `toString` 方法没有重写的话，会返回 `[Object type]`，其中 type 为对象的类型。但当除了 Object 类型的对象外，其他类型直接使用 `toString` 方法时，会直接返回都是内容的字符串，所以我们需要使用call或者apply方法来改变toString方法的执行上下文。

```
const an = ['Hello','An'];
an.toString(); // "Hello,An"
Object.prototype.toString.call(an); // "[object Array]"
复制代码
```

这种方法对于所有基本的数据类型都能进行判断，即使是 null 和 undefined 。

```
Object.prototype.toString.call('An') // "[object String]"
Object.prototype.toString.call(1) // "[object Number]"
Object.prototype.toString.call(Symbol(1)) // "[object Symbol]"
Object.prototype.toString.call(null) // "[object Null]"
Object.prototype.toString.call(undefined) // "[object Undefined]"
Object.prototype.toString.call(function(){}) // "[object Function]"
Object.prototype.toString.call({name: 'An'}) // "[object Object]"
复制代码
```

`Object.prototype.toString.call()` 常用于判断浏览器内置对象。

#### 2. instanceof

`instanceof`  的内部机制是通过判断对象的原型链中是不是能找到类型的 `prototype`。

使用 `instanceof`判断一个对象是否为数组，`instanceof` 会判断这个对象的原型链上是否会找到对应的 `Array` 的原型，找到返回 `true`，否则返回 `false`。

```
[]  instanceof Array; // true
复制代码
```

但 `instanceof` 只能用来判断对象类型，原始类型不可以。并且所有对象类型 instanceof Object 都是 true。

```
[]  instanceof Object; // true
```

#### 3. Array.isArray()

- 功能：用来判断对象是否为数组

- instanceof 与 isArray

  当检测Array实例时，`Array.isArray` 优于 `instanceof` ，因为 `Array.isArray` 可以检测出 `iframes`

  ```js
  var iframe = document.createElement('iframe');
  document.body.appendChild(iframe);
  xArray = window.frames[window.frames.length-1].Array;
  var arr = new xArray(1,2,3); // [1,2,3]

  // Correctly checking for Array
  Array.isArray(arr);  // true
  Object.prototype.toString.call(arr); // true
  // Considered harmful, because doesn't work though iframes
  arr instanceof Array; // false
  ```

- `Array.isArray()` 与 `Object.prototype.toString.call()`

  `Array.isArray()`是ES5新增的方法，当不存在 `Array.isArray()` ，可以用 `Object.prototype.toString.call()` 实现。

  ```
  if (!Array.isArray) {
    Array.isArray = function(arg) {
      return Object.prototype.toString.call(arg) === '[object Array]';
    };
  }
  ```

####  重绘

由于节点的几何属性发生改变或者由于样式发生改变而不会影响布局的，称为重绘，例如`outline`, `visibility`, `color`、`background-color`等，重绘的代价是高昂的，因为浏览器必须验证DOM树上其他节点元素的可见性。

#### 回流

回流是布局或者几何属性需要改变就称为回流。回流是影响浏览器性能的关键因素，因为其变化涉及到部分页面（或是整个页面）的布局更新。一个元素的回流可能会导致了其所有子元素以及DOM中紧随其后的节点、祖先节点元素的随后的回流。

```
<body>
<div class="error">
    <h4>我的组件</h4>
    <p><strong>错误：</strong>错误的描述…</p>
    <h5>错误纠正</h5>
    <ol>
        <li>第一步</li>
        <li>第二步</li>
    </ol>
</div>
</body>
复制代码
```

在上面的HTML片段中，对该段落(`<p>`标签)回流将会引发强烈的回流，因为它是一个子节点。这也导致了祖先的回流（`div.error`和`body` – 视浏览器而定）。此外，`<h5>`和`<ol>`也会有简单的回流，因为这些节点在DOM中回流元素之后。**大部分的回流将导致页面的重新渲染。**

**回流必定会发生重绘，重绘不一定会引发回流。**

#### 减少重绘与回流

1. CSS
   - **使用 transform 替代 top**
   - **使用 visibility 替换 display: none** ，因为前者只会引起重绘，后者会引发回流
   - **避免使用table布局**，可能很小的一个小改动会造成整个 `table` 的重新布局。
   - **尽可能在DOM树的最末端改变class**，回流是不可避免的，但可以减少其影响。尽可能在DOM树的最末端改变class，可以限制了回流的范围，使其影响尽可能少的节点。
   - **避免设置多层内联样式**，CSS 选择符**从右往左**匹配查找，避免节点层级过多。

- **将动画效果应用到position属性为absolute或fixed的元素上**，避免影响其他元素的布局，这样只是一个重绘，而不是回流，同时，控制动画速度可以选择 `requestAnimationFrame`，详见[探讨 requestAnimationFrame](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FLuNaHaiJiao%2Fblog%2Fissues%2F30)。
- **避免使用CSS表达式**，可能会引发回流。
- **将频繁重绘或者回流的节点设置为图层**，图层能够阻止该节点的渲染行为影响别的节点，例如`will-change`、`video`、`iframe`等标签，浏览器会自动将该节点变为图层。
- **CSS3 硬件加速（GPU加速）**，使用css3硬件加速，可以让`transform`、`opacity`、`filters`这些动画不会引起回流重绘 。但是对于动画的其它属性，比如`background-color`这些，还是会引起回流重绘的，不过它还是可以提升这些动画的性能。

2. JavaScript

- **避免频繁操作样式**，最好一次性重写`style`属性，或者将样式列表定义为`class`并一次性更改`class`属性。
- **避免频繁操作DOM**，创建一个`documentFragment`，在它上面应用所有`DOM操作`，最后再把它添加到文档中。
- **避免频繁读取会引发回流/重绘的属性**，如果确实需要多次使用，就用一个变量缓存起来。
- **对具有复杂动画的元素使用绝对定位**，使它脱离文档流，否则会引起父元素及后续元素频繁回流。

### js为什么是单线程语言，怎么模拟多线程

- JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？
- 为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质。

### 评价一下三种方式实现继承的有缺点并改进；

```
function Animal(){}
function Cat(){}
方法1：
Cat.prototype = new Animal();方法2：
Cat.ptototype = Animal.prototype;方法3：
Cat.prototype = Object.create(Animal.prototype);复制代码
```

**方法1：**

- **优点**

1. 正确设置原型链实现继承
2. 父类实例属性得到继承，原型链查找效率提高，也能为一些属性提供合理的默认值

- **缺点**

1. 父类实例属性为引用类型时，不恰当地修改会导致所有子类被修改 
2. 无法传递参数

**方法二：**

- **优点**

1. 正确设置原型链实现继承

- **缺点**

1. 父类构造函数原型与子类相同。修改子类原型添加方法会修改父类

**方法三：**

- **优点**

1. 正确设置原型链且避免方法1.2中的缺点

- **缺点**

1. ES5方法需要注意兼容性

**改进：**

- 所有三种方法应该在子类构造函数中调用父类构造函数实现实例属性初始化

```
function Cat() {    
    Animal.call(this);
}复制代码
```

- 封装一个原型继承的方法

```
function extend(Child, Parent) {
    var F = function(){};
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    //用新创建的对象替代子类默认原型，设置Rect.prototype.constructor = Rect;保证一致性
    Child.prototype.constructor = Child;  
}
```

