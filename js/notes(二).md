#### Object.defineProperty的用法详解

该方法是es5的方法（千万不要以为是es6的哦），作用是直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。（**切记只能用在对象身上不能用在数组身上**）

###### 1、语法

```
Object.defineProperty(obj, prop, descriptor)
```

###### 2、参数说明：

- obj：必需。目标对象 
- prop：必需。需定义或修改的属性的名字
- descriptor：必需。目标属性所拥有的特性

###### 3、属性使用案例

修改某个属性的值时，给这个属性添加一些特性。

```
let person = {}; 
Object.defineProperty(person, 'name', {   
    writable: true || false,   
    configurable: true || false,   
    enumerable: true || false,  
    value:'gjf' 
});
```

属性详解：

- writable：是否可以被重写，true可以重写，false不能重写，默认为false。
- enumerable：是否可以被枚举（使用for...in或Object.keys()）。设置为true可以被枚举；设置为false，不能被枚举。默认为false。
- value：值可以使任意类型的值，默认为undefined
- configurable：是否可以删除目标属性或是否可以再次修改属性的特性（writable, configurable, enumerable）。设置为true可以被删除或可以重新设置特性；设置为false，不能被可以被删除或不可以重新设置特性。默认为false。

###### 4、存取器描述（get和set）

- **注意：当使用了getter或setter方法，不允许使用writable和value这两个属性**

```
let person = {};
let n = 'gjf';
Object.defineProperty(person, 'name', { 
    configurable: true,  
    enumerable: true, 
    get() {    
        //当获取值的时候触发的函数    
        return n  
    },  
    set(val) {    
        //当设置值的时候触发的函数,设置的新值通过参数val拿到    
        n = val;  
    }
});
console.log(person.name) //gjf
person.name = 'newGjf'
console.log(person.name) //newGif
```

#### CSS3新增伪类有那些？

```
  	举例：
  	p:first-of-type	选择属于其父元素的首个 <p> 元素的每个 <p> 元素。
  	p:last-of-type	选择属于其父元素的最后 <p> 元素的每个 <p> 元素。
      p:only-of-type	选择属于其父元素唯一的 <p> 元素的每个 <p> 元素。
  	p:only-child		选择属于其父元素的唯一子元素的每个 <p> 元素。
  	p:nth-child(2)	选择属于其父元素的第二个子元素的每个 <p> 元素。

  	::after			在元素之前添加内容,也可以用来做清除浮动。
  	::before			在元素之后添加内容
    :enabled  		
  	:disabled 		控制表单控件的禁用状态。
  	:checked        单选框或复选框被选中。
```

#### CSS3有哪些新特性？

```
    新增各种CSS选择器	（: not(.input)：所有 class 不是“input”的节点）
    圆角		    （border-radius:8px）
    多列布局	    （multi-column layout）
    阴影和反射	（Shadow\Reflect）
    文字特效		（text-shadow、）
    文字渲染		（Text-decoration）
    线性渐变		（gradient）
    旋转		 	（transform）
    缩放,定位,倾斜,动画,多背景
    例如:transform:\scale(0.85,0.90)\ translate(0px,-30px)\ skew(-9deg,0deg)\Animation:
```

#### html5有哪些新特性、移除了那些元素？如何处理HTML5新标签的浏览器兼容问题？如何区分 HTML 和 HTML5？

```
  * HTML5 现在已经不是 SGML 的子集，主要是关于图像，位置，存储，多任务等功能的增加。
  	  绘画 canvas;
  	  用于媒介回放的 video 和 audio 元素;
  	  本地离线存储 localStorage 长期存储数据，浏览器关闭后数据不丢失;
        sessionStorage 的数据在浏览器关闭后自动删除;
  	  语意化更好的内容元素，比如 article、footer、header、nav、section;
  	  表单控件，calendar、date、time、email、url、search;
  	  新的技术webworker, websocket, Geolocation;

    移除的元素：
  	  纯表现的元素：basefont，big，center，font, s，strike，tt，u;
  	  对可用性产生负面影响的元素：frame，frameset，noframes；

  * 支持HTML5新标签：
  	 IE8/IE7/IE6支持通过document.createElement方法产生的标签，
    	 可以利用这一特性让这些浏览器支持HTML5新标签，
    	 浏览器支持新标签后，还需要添加标签默认的样式。

       当然也可以直接使用成熟的框架、比如html5shim;
  	 <!--[if lt IE 9]>
  		<script> src="http://html5shim.googlecode.com/svn/trunk/html5.js"</script>
  	 <![endif]-->

  * 如何区分HTML5： DOCTYPE声明\新增的结构元素\功能元素
```