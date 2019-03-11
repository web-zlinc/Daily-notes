## webpack4初体验

#### 安装

```
// webpack4中除了正常安装webpack之外，需要再单独安一个webpack-cli
npm i webpack webpack-cli -D
```

★ npm i -D 是 npm install --save-dev 的简写，是指安装模块并保存到 package.json 的 devDependencies中，主要在开发环境中的依赖包

#### webpack是基于Node的

在项目下创建一个webpack.config.js(默认，可修改)文件来配置webpack

```
module.exports = {
    entry: '',               // 入口文件
    output: {},              // 出口文件
    module: {},              // 处理对应模块
    plugins: [],             // 对应的插件
    devServer: {},           // 开发服务器配置
    mode: 'development'      // 模式配置
}
```

★ 启动devServer需要安装一下webpack-dev-server

```
npm i webpack-dev-server -D
```

从0开始去写配置

```
// webpack.config.js
const path = require('path');
module.exports = {
    entry: './src/index.js',    // 入口文件
    output: {
        filename: 'bundle.js',      // 打包后的文件名称
        path: path.resolve('dist')  // 打包后的目录，必须是绝对路径
    }
}
```

打包一下看看

![img](https://user-gold-cdn.xitu.io/2018/4/26/16300951712e1f7a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 配置执行文件

**npm run dev**这样的命令，既然是通过npm执行的命令，我们就应该找到package.json里的执行脚本去配置一下命令，这里如下图所示

![img](https://user-gold-cdn.xitu.io/2018/4/25/162fd57a53a0e108?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**npm run build**就是我们打包后的文件，这是生产环境下，上线需要的文件

**npm run dev**是我们开发环境下打包的文件，当然由于devServer帮我们把文件放到内存中了，所以并不会输出打包后的dist文件夹

通过**npm run build**之后会生成一个dist目录文件夹，就和上面打包后的样子一样了

#### 多入口文件

有两种实现方式进行打包

- 一种是没有关系的但是要打包到一起去的，可以写一个数组，实现多个文件打包
- 另一种就是每一个文件都单独打包成一个文件的
- 下面就来看看这两种方式的写法

```
let path = require('path');

module.exports = {
    // 1.写成数组的方式就可以打出多入口文件，不过这里打包后的文件都合成了一个
    // entry: ['./src/index.js', './src/login.js'],
    // 2.真正实现多入口和多出口需要写成对象的方式
    entry: {
        index: './src/index.js',
        login: './src/login.js'
    },
    output: {
        // 1. filename: 'bundle.js',
        // 2. [name]就可以将出口文件名和入口文件名一一对应
        filename: '[name].js',      // 打包后会生成index.js和login.js文件
        path: path.resolve('dist')
    }
}
```

npm run build后，会生成打包好的两个js文件，如图所示

![img](https://user-gold-cdn.xitu.io/2018/4/26/16300af9d7802153?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 配置Html模板

使用的时候不能在dist目录下去创建一个html文件，然后去引用打包后的js吧

我们需要实现html打包功能，可以通过一个模板实现打包出引用好路径的html来

需要用到一个常用的插件了，**html-webpack-plugin**，用之前我们来安一下它

```
npm i html-webpack-plugin -D
```

因为是个插件，所以需要在config.js里引用一下的

```
let path = require('path');
// 插件都是一个类，所以我们命名的时候尽量用大写开头
let HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    output: {
        // 添加hash可以防止文件缓存，每次都会生成4位的hash串
        filename: 'bundle.[hash:4].js',   
        path: path.resolve('dist')
    },
    plugins: [
        // 通过new一下这个类来使用插件
        new HtmlWebpackPlugin({
            // 用哪个html作为模板
            // 在src目录下创建一个index.html页面当做模板来用
            template: './src/index.html',
            hash: true, // 会在打包好的bundle.js后面加上hash串
        })
    ]
}
```

再npm run build打包看一下

![img](https://user-gold-cdn.xitu.io/2018/4/26/16300c7f0dd28ade?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 多页面开发，怎么配置多页面

如果开发的时候不只一个页面，需要配置多页面，html-webpack-plugin插件自有办法，

```
let path = require('path');
let HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    // 多页面开发，怎么配置多页面
    entry: {
        index: './src/index.js',
        login: './src/login.js'
    },
    // 出口文件  
    output: {                       
        filename: '[name].js',
        path: path.resolve('dist')
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',   
            filename: 'index.html',
            chunks: ['index']   // 对应关系,index.js对应的是index.html
        }),
        new HtmlWebpackPlugin({
            template: './src/login.html',
            filename: 'login.html',
            chunks: ['login']   // 对应关系,login.js对应的是login.html
        })
    ]
}
```

npm run build看打包后的样子 

![img](https://user-gold-cdn.xitu.io/2018/4/26/16300d9f23bccd0e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 引用css文件

css样式的loader

```
npm i style-loader css-loader -D
// 引入less文件的话，也需要安装对应的loader
npm i less less-loader -D
```

配置css文件的解析

```
// index.js
import './css/style.css';   // 引入css
import './less/style.less'; // 引入less

console.log('这里是打包文件入口-index.js');

// webpack.config.js
module.exports = {
    entry: {
        index: './src/index.js'
    },
    output: {
        filename: 'bundle.js',
        path: path.resolve('dist')
    },
    module: {
        rules: [
            {
                test: /\.css$/,     // 解析css
                use: ['style-loader', 'css-loader'] // 从右向左解析
                /* 
                    也可以这样写，这种方式方便写一些配置参数
                    use: [
                        {loader: 'style-loader'},
                        {loader: 'css-loader'}
                    ]
                */
            }
        ]
    }
}
复制代码
```



![img](https://user-gold-cdn.xitu.io/2018/4/26/16300f20c7d6587c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- 此时打包后的css文件是以行内样式style的标签写进打包后的html页面中，如果样式很多的话，希望直接用link的方式引入进去，这时候需要把**css拆分出来**
- **extract-text-webpack-plugin**插件相信用过的人都知道它是干什么的，它的功效就在于会将打包到js里的css文件进行一个拆分

#### 拆分CSS

```
// @next表示可以支持webpack4版本的插件
npm i extract-text-webpack-plugin@next -D
```

```
let path = require('path');
let HtmlWebpackPlugin = require('html-webpack-plugin');
// 拆分css样式的插件
let ExtractTextWebpackPlugin = require('extract-text-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    output: {
        filaneme: 'bundle.js',
        path: path.resolve('dist')
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ExtractTextWebpackPlugin.extract({
                    // 将css用link的方式引入就不再需要style-loader了
                    use: 'css-loader'       
                })
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
        }),
        // 拆分后会把css文件放到dist目录下的css/style.css
        new ExtractTextWebpackPlugin('css/style.css')  
    ]
}
```

打包的html页面就以link的方式去引入css

![img](https://user-gold-cdn.xitu.io/2018/4/26/16300fa1d7d47ebd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 拆分成多个css

```
// 正常写入的less
let styleLess = new ExtractTextWebpackPlugin('css/style.css');
// reset
let resetCss = new ExtractTextWebpackPlugin('css/reset.css');

module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: resetCss.extract({
                    use: 'css-loader'
                })
            },
            {
                test: /\.less$/,
                use: styleLess.extract({
                    use: 'css-loader'
                })
            }
        ]
    },
    plugins: [
        styleLess,
        resetCss
    ]
}
复制代码
```

通过这样操作后可以打包成两个不同的css文件，如下图 

![img](https://user-gold-cdn.xitu.io/2018/4/28/1630b17eb24419be?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



#### 引用图片

```
npm i file-loader url-loader -D
复制代码
```

如果是在css文件里引入的如背景图之类的图片，就需要指定一下相对路径

```
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ExtractTextWebpackPlugin.extract({
                    use: 'css-loader',
                    publicPath: '../'
                })
            },
            {
                test: /\.(jpe?g|png|gif)$/,
                use: [
                    {
                        loader: 'url-loader',
                        options: {
                            limit: 8192,    // 小于8k的图片自动转成base64格式，并且不会存在实体图片
                            outputPath: 'images/'   // 图片打包后存放的目录
                        }
                    }
                ]
            }
        ]
    }
}
```

在css中指定了publicPath路径这样就可以根据相对路径引用到图片资源了，如下图所示 

![img](https://user-gold-cdn.xitu.io/2018/4/26/163015e474789835?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 页面img引用图片

页面中经常会用到img标签，img引用的图片地址也需要一个loader来帮我们处理好

```
npm i html-withimg-loader -D
复制代码
module.exports = {
    module: {
        rules: [
            {
                test: /\.(htm|html)$/,
                use: 'html-withimg-loader'
            }
        ]
    }
}
```

这样再打包后的html文件下img就可以正常引用图片路径了 

![img](https://user-gold-cdn.xitu.io/2018/4/26/1630161fbaa1d225?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 引用字体图片和svg图片

字体图标和svg图片都可以通过file-loader来解析

```
module.exports = {
    module: {
        rules: [
            {
                test: /\.(eot|ttf|woff|svg)$/,
                use: 'file-loader'
            }
        ]
    }
}
```

这样即使样式中引入了这类格式的图标或者图片都没有问题了，img如果也引用svg格式的话，配合上面写好的html-withimg-loader就都没有问题了

#### 添加CSS3前缀

通过postcss中的autoprefixer可以实现将CSS3中的一些需要兼容写法的属性添加响应的前缀

```
npm i postcss-loader autoprefixer -D
复制代码
```

安装后，我们还需要像webpack一样写一个config的配置文件，在项目根目录下创建一个**postcss.config.js**文件，配置如下：

```
module.exports = {
    plugins: [require('autoprefixer')]  // 引用该插件即可了
}
```

然后在webpack里配置postcss-loader

```
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ['style-loader', 'css-loader', 'postcss-loader']
            }
        ]
    }
}
```

#### 转义ES6

```
npm i babel-core babel-loader babel-preset-env babel-preset-stage-0 -D
```

要兼容的代码不仅仅包含ES6还有之后的版本和那些仅仅是草案的内容，所以我们可以通过一个.babelrc文件来配置一下

```
// .babelrc
{
"presets": ["env", "stage-0"]   // 从右向左解析
}
```

在webpack里配置一下babel-loader既可以做到代码转成ES5了

```
module.exports = {
    module: {
        rules: [
            {
                test:/\.js$/,
                use: 'babel-loader',
                include: /src/,          // 只转化src目录下的js
                exclude: /node_modules/  // 排除掉node_modules，优化打包速度
            }
        ]
    }
}
```

每次打包之前将dist目录下的文件都清空，然后再把打好包的文件放进去

这里提供一个clean-webpack-plugin插件

```
npm i clean-webpack-plugin -D
```

```
let CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
    plugins: [
        // 打包前先清空
        new CleanWebpackPlugin('dist')  
    ]
}
```

#### 启动静态服务器

启动一个静态服务器，默认会**自动刷新**，对html,css,js文件做了修改并保存后，浏览器会默认刷新一次展现修改后的效果

如果devServer里open设为true后，会自动打开浏览器

```
module.exports = {
    devServer: {
        contentBase: './dist',
        host: 'localhost',      // 默认是localhost
        port: 3000,             // 端口
        open: true,             // 自动打开浏览器
        hot: true               // 开启热更新
    }
}
```

当然在npm run dev命令下，打包的文件存在于内存中，并不会产生在dist目录下

#### 热更新和自动刷新的区别

在配置devServer的时候，如果hot为true，就代表开启了热更新

并没那么简单，因为热更新还需要配置一个webpack自带的插件并且还要在主要js文件里检查是否有module.hot

```
// webpack.config.js
let webpack = require('webpack');

module.exports = {
    plugins: [
        // 热更新，热更新不是刷新
        new webpack.HotModuleReplacementPlugin()
    ],
    devServer: {
        contentBase: './dist',
        hot: true,
        port: 3000
    }
}

// 此时还没完虽然配置了插件和开启了热更新，但实际上并不会生效

// index.js
let a = 'hello world';
document.body.innerHTML = a;
console.log('这是webpack打包的入口文件');

// 还需要在主要的js文件里写入下面这段代码
if (module.hot) {
    // 实现热更新
    module.hot.accept();
}
```

#### resolve解析

在webpack的配置中，resolve我们常用来配置别名和省略后缀名

```
module.exports = {
    resolve: {
        // 别名
        alias: {
            $: './src/jquery.js'
        },
        // 省略后缀
        extensions: ['.js', '.json', '.css']
    },
}
```

#### 提取公共代码

在webpack4之前，提取公共代码通过CommonsChunkPlugin的插件来办到的。到了4以后，内置了一个一模一样的功能，而且起了一个好听的名字叫“优化”

来看看如何提取公共代码

```
// 假设a.js和b.js都同时引入了jquery.js和一个写好的utils.js
// a.js和b.js
import $ from 'jquery';
import {sum} from 'utils';
```

两个js中其中公共部分的代码就是jquery和utils里的代码了

可以针对第三方插件和写好的公共文件

```
module.exports = {
    entry: {
        a: './src/a.js',
        b: './src/b.js'
    },
    output: {
        filename: '[name].js',
        path: path.resolve('dust')
    },
    // 提取公共代码
+   optimization: {
        splitChunks: {
            cacheGroups: {
                vendor: {   // 抽离第三方插件
                    test: /node_modules/,   // 指定是node_modules下的第三方包
                    chunks: 'initial',
                    name: 'vendor',  // 打包后的文件名，任意命名    
                    // 设置优先级，防止和自定义的公共代码提取时被覆盖，不进行打包
                    priority: 10    
                },
                utils: { // 抽离自己写的公共代码，utils这个名字可以随意起
                    chunks: 'initial',
                    name: 'utils',  // 任意命名
                    minSize: 0    // 只要超出0字节就生成一个新包
                }
            }
        }
+   },
    plugins: [
        new HtmlWebpackPlugin({
            filename: 'a.html',
            template: './src/index.html',  // 以index.html为模板
+           chunks: ['vendor', 'a']
        }),
        new HtmlWebpackPlugin({
            filename: 'b.html',
            template: './src/index.html',  // 以index.html为模板
+           chunks: ['vendor', 'b']
        })
    ]
}
```

通过以上配置，可以把引入到a.js和b.js中的这部分公共代码提取出来，如下图所示 

![img](https://user-gold-cdn.xitu.io/2018/4/27/16302c4bc8dd3596?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



#### 指定webpack配置文件

在package.json的脚步里，我们可以配置调用不同的webpack配置文件进行打包，举个栗子

![img](https://user-gold-cdn.xitu.io/2018/4/27/16302e1dd30d1f06?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 这样的好处在于可以针对不同的需求进行一个特定的打包配置

