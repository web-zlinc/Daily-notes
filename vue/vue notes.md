## vue interview

#### 对MVVM的理解？

MVVM是Model-View-ViewModel的缩写，Model代表数据模型负责业务逻辑和数据封装，View代表UI组件负责界面和显示，ViewModel监听模型数据的改变和控制视图行为，处理用户交互，简单来说就是通过双向数据绑定把View层和Model层连接起来。在MVVM架构下，View和Model没有直接联系，而是通过ViewModel进行交互，我们只关注业务逻辑，不需要手动操作DOM，不需要关注View和Model的同步工作。

#### route和router的区别

route是路由信息对象，包括path,params,hash,query,fullPath,matched,name等路由信息参数。

router是路由实例对象，包括了路由的跳转方法，钩子函数。

#### 对keep-alive的了解

keep-alive是一个内置组件，可使被包含的组件保留状态或避免重新渲染，有include（包含的组件缓存）和exclude（排除的组件不缓存）两个属性。

#### 组件中data什么时候可以适用对象

组件复用时所有组件实例都会共享data，如果data是对象就会造成一个组件修改data以后会影响到其他所有组件，所以需要将data写成函数，每次用到就调用一次函数获得新的数据

当我们使用new Vue()的方式的时候，无论我们将data设置为对象还是函数都是可以的，因为new Vue()的方式是生成一个根组件，该组件不会复用，也就不存在共享data的情况。

#### computed和watch区别

computed是计算属性，依赖其他属性计算值，并且computed的值有缓存，只有当计算值变化才会返回内容

watch监听到值的变化就会执行回调，在回调中可以进行一些逻辑操作。

一般来说需要依赖别的属性来动态获得值的时候可以使用computed，对于监听到值的变化需要做一些复杂业务逻辑的情况可以使用watch。

```
data: {
    firstName: 'Chen',
    lastName: 'Miao',
    fullName: 'Chen Miao'
},
watch: {
    firstName: function(val){
        this.fullName = val+ ' '+this.lastName
    },
    lastName: function(val){
        this.fullName = this.firstName+ ' '+val
    }
},
computed: {
    anoFullName: function(){
        return this.firstName+' '+this.lastName
    }
}
```

#### mixin和mixins区别

mixin用于全局混入，会影响到每个组件实例，通常插件都是这样做初始化的。

```
Vue.mixin({
    beforeCreate(){
        // 会影响到每个组件的beforeCreate钩子函数
    }
})
```

mixins最常用的扩展组件的方式。如果多个组件有相同的业务逻辑，就可将这些逻辑剥离出来，通过mixins混入代码。需要注意：mixins混入的钩子函数会先于组件内的钩子函数执行，并且在遇到同名选项的时候也会有选择性的进行合并。

#### 如何使用vue.nextTick()？

nextTick可以使我们在下次DOM更新循环结束之后执行延迟回调，用于获得更新后的DOM

```
data:function(){
    return {
        message: '没有更新'
    }
},
methods: {
    updateMessage: function(){
        this.message='更新完成'
        console.log(this.$el.textContent) // '没有更新'
        this.$nextTick(function(){
          console.log(this.$el.textContent)// '更新完成'  
        })
    }
}
```

#### 组件通信

**父子通信**

#### 1.props和emit

父组件通过props传递数据给子组件，子组件通过emit发送事件传递给父组件。

#### 2.v-model

v-model其实是props,emit的语法糖，v-model默认会解析成名为value的prop和名为input的事件。

```
// 父组件
<children v-model="msg"></children>
<p>{{msg}}</p>

data(){
    return{
        msg:'model'
    }
}
// 子组件
<input :value="value" @input="toInput" />

props: ['value'],
methods: {
    toInput(e){
        this.$emit('input', e.target.value)
    }
}
```

#### 3.在父组件使用`$children`访问子组件，在子组件中使用`$parent`访问父组件。

#### `$listeners`和`$attrs`

`$attrs`--继承所有父组件属性（除了prop传递的属性）

inheritAttrs--默认值true，继承所有父组件属性（除props），为true会将attrs中的属性当做html的data属性渲染到dom根节点上

`$listeners`--属性，包含了作用在这个组件上所有监听器，v-on="$listeners"将所有事件监听器指向这个组件的某个特定子元素

```
// 父组件
<children :child1="child1" :child2="child2" @test1="onTest1"
@test2="onTest2"></children>

data(){
    return {
        child1: 'childOne',
        child2: 'childTwo'
    }
},
methods: {
    onTest1(){
        console.log('test1 running')
    },
    onTest2(){
        console.log('test2 running')
    }
}

// 子组件
<p>{{child1}}</p>
<child v-bind="$attrs" v-on="$listeners"></child>

props: ['child1'],
mounted(){
    this.$emit('test1')
}

// 孙组件
<p>{{child2}</p>
<p>{{$attrs}}</p>

props: ['child2'],
mounted(){
    this.$emit('test2')
}
```

#### .sync方式

在vue1.x中是对prop进行双向绑定，在vue2只允许单向数据流，也是一个语法糖

```
// 父组件
<child :count.sync="num" />

data(){
    return {
        num: 0
    }
}
// 子组件
<div @click="handleAdd">add</div>

data(){
    return {
        counter: this.count
    }
},
props: ["count"],
methods: {
    handleAdd(){
        this.$emit('update:count', ++this.counter)
    }
}
```

### 跨多层次组件通信

可以使用provide/inject，虽然文档中不推荐直接使用在业务中。

假设有父组件A，然后有一个跨多层次的子组件B

```
// 父组件A
export default{
    provide: {
        data: 1
    }
}
// 子组件B
export default{
    inject: ['data'],
    mounted(){
        // 无论跨几层都能获取父组件的data属性
        console.log(this.data); // 1
    }
}
```

#### eventBus的使用

1.新建一个bus.js文件

```
import Vue from 'vue';
export default new Vue();
```

2.使用它

```
<div @click="addCart">添加</div>
import Bus from 'bus.js';
export default{
    methods: {
        addCart(event){
            Bus.$emit('getTarget', event.target)
        }
    }
}
// 另一组件
export default{
    created(){
        Bus.$on('getTarget', target =>{
            console.log(target)
        })
    }
}
```

### vue-router路由

单页面应用SPA的核心之一是：**更新视图而不重新请求页面**

###### 普通路由

`router.push('home')`

`router.push({path: 'home')`

###### 命名路由

```
const router=new VueRouter({
    routes: [{
        path: '/user',
        name: 'user',
        component: User
    }]
})
```

```
<router-link :to="{name: 'user'}"></router-link>
```

```
router.push({
    name: 'user'
})
```

###### 动态路由匹配

```
<div>{{$route.params.id}}</div>
```

```
const router = new VueRouter({
    routes: [{
        path: '/user/:id',
        component: User
    }]
})
```

`router.push({name:'user',params: {id: 123})`

###### 嵌套路由

```
<h2>{{$route.params.id}}</h2>
<router-view></router-view>
```

```
const router = new VueRouter({
    routes: [{
        path: '/user/:id',
        children: [{
            // 当/user/:id
            path: '',
            component: UserHome
        },{
            // 当/user/:id/profile
            path: 'profile',
            component: UserProfile
        }]
    }]
})
```

###### 参数的路由

`router.push({path:'register',query:{plan:'private'})`

###### 编程式导航

#### router.push()

参数：

1.字符串

router.push('home')

2.对象

router.push({path: 'home'})

3.命名的路由

router.push({name: 'user',params: {userId: 123}})

4.带查询参数，变成/register?plan=private

router.push({path: 'register',query: {plan: 'private'}})

#### router.replace()

不会向history添加新纪录，替换当前的history记录

点击`<router-link :to="..." replace>`等同于调用`router.replace(...)`

#### router.go(n)

在历史记录中向前或向后退多少步

```
// 前进一步，等同history.forward()
router.go(1)
// 后退一步
router.go(-1)
// 前进3步记录
router.go(3)
```

###### 命名视图

```
<router-view></router-view>
<router-view name="a"></router-view>
<router-view name="b"></router-view>
```

```
const router = new VueRouter({
    routes: [{
        path: '/',
        components: {
            default: Foo,
            a: Bar,
            b: Baz
        }
    }]
})
```

#### vue路由的钩子函数

导航钩子主要用来拦截导航，让它完成跳转或取消。

router.beforEach((to,from,next) => {})

钩子是异步执行解析的，每个钩子方法接收三个参数：

to: Route即将进入的目标路由对象

from: Route当前导航正要离开的路由

next: Function，调用该方法来resolve这个钩子，执行效果看参数

- next():进行下一个钩子
- next(false):中断当前的导航
- next('/')或next({path: '/'}):跳转到另一地址
