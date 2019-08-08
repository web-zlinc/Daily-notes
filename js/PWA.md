# PWA

全名，progressive web app： 渐进式网页应用。是谷歌推出的，这样理解：

- 我们一般写 web 应用，在 pc 上是没有缓存的，打开页面的时去请求数据。
- 第二个也没有像 app 一样的小图标放在桌面，一点开就进入了应用，而是通过打开浏览器输入网址，
- 第三个就是，不能像 app 一样给用户推送消息，像微博会跟你推送说有谁评论了你的微博之类的功能。

**谷歌推出的 pwa，具有这些了这些特点， 使我们的 web 应用，能够像一款 app 一样使用。并且对比与 app, 它不用复杂的安装，也不用下载更新包，刷新页面就可以了(注意到缓存的处理)。**

#### **怎么实现这些功能的呢**

**关于缓存**

其实这个就是 我们平时做的 Session 啊、localStorage、CacheStorage 之类的。

这里用的是 [cacheStorage](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FCacheStorage) 缓存，它提供了一个ServiceWorker类型的工作者或window范围可以访问的所有命名缓存的主目录, 并维护字符串的映射名称到相应的 Cache 对象。 主要方法包括

![这里写图片描述](https://user-gold-cdn.xitu.io/2018/3/6/161fb4eccdc2700f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

有了这些方法你可以对你的缓存进行操作。目前还在草案状态，仅火狐和谷歌浏览器支持此特性。

PWA是通过 ServiceWorker 访问 cache ,所以需要注册 ServiceWorker 工作者。在之前别忘记判断浏览器是否支持。

```
if ('serviceWorker' in navigator) {
	navigator.serviceWorker.register(sw.js) // 注册sw.js 文件中变成的服务对象，返回注册成功的对象
	.then(function(swReg){
          swRegistration = swReg;
     }).catch(function(error) {
          console.error('Service Worker Error', error);
     });
}
```

[Service Worker](https://link.juejin.im?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FAPI%2FService_Worker_API) **相当于浏览器和网络之间的代理服务器**，**可以拦截网络请求，做一些你可能需要的处理(请求资源从缓存中获取等)。**

- 它能够创建有效的离线体验，拦截网络请求，并根据网络是否可用判断是否使用缓存数据或者更新缓存数据。
- 它们还允许访问推送的通知和后台的API。

关于 sw.js 中具体的缓存的代码：

```
'use strict'
let cacheName = 'pwa-demo-assets'; // 缓存名字
let imgCacheName = 'pwa-img';
let filesToCache;
filesToCache = [ // 所需缓存的文件
    '/',
    '/index.html',
    '/scripts/app.js',
    '/assets/imgs/48.png',
    '/assets/imgs/96.png',
    '/assets/imgs/192.png',
    '/dist/js/app.js',
    '/manifest.json'
];

self.addEventListener('install', function(e) {
    e.waitUntil(
	    // 安装服务者时，对需要缓存的文件进行缓存
        caches.open(cacheName).then(function(cache) {
            return cache.addAll(filesToCache);
        })
    );
});


self.addEventListener('fetch', (e) => {
    // 判断地址是不是需要实时去请求，是就继续发送请求
    if (e.request.url.indexOf('/api/400/200') > -1) {
        e.respondWith(
            caches.open(imgCacheName).then(function(cache){
                 return fetch(e.request).then(function (response){
                    cache.put(e.request.url, response.clone()); // 每请求一次缓存更新一次新加载的图片
                    return response;
                });
            })
        );
    } else {
        e.respondWith(
	        // 匹配到缓存资源，就从缓存中返回数据
            caches.match(e.request).then(function (response) {
                return response || fetch(e.request);
            })
        );
    }

});
```

**这里进而就引入到 pwa 的推送通知功能。这都是通过 ServiceWorker 去实现的。**

基本原理是，你的客户端要和推送服务进行绑定，会生成一个绑定后的推送服务 API 接口，服务端调用此接口，发送消息。同时，浏览器也要支持推送功能，在注册 sw 时, 加上推送功能的判断。

```
if ('serviceWorker' in navigator && 'PushManager' in window) {
	navigator.serviceWorker.register(sw.js)
	.then(function(swReg) {
        swRegistration = swReg;
    }).catch(function(error) {
        console.error('Service Worker Error', error);
        });
 } else {
     console.warn('Push messaging is not supported');
 }
```

PushManager 注册好之后， 那么要做的就是浏览器和服务器的绑定了。



![这里写图片描述](https://user-gold-cdn.xitu.io/2018/3/6/161fb4eccdb7fa01?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 此图是用户订阅某个应用程序的推送服务。 客户端传入应用程序服务器公钥，向将生成端点的 `webpush 服务器`





![这里写图片描述](https://user-gold-cdn.xitu.io/2018/3/6/161fb4ecce21ca23?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 服务器发送推送的时候,请求相关接口，验证成功后推送服务会发消息给客户端。



**最后关于桌面小图标**

这个可以说是非常简单了，就是一个 manifest.json 配置文件，然后在页面引入此文件就好了

```
<!-- 加载清单 -->
<link rel="manifest" href="./manifest.json">
```

关于[清单内容](https://link.juejin.im?target=https%3A%2F%2Fdevelopers.google.com%2Fweb%2Ffundamentals%2Fweb-app-manifest%2F)这里简单介绍一下：

```
{
    "short_name": "pwa",
    "name": "pwa - demo", // 应用名称
    "icons": [ // 应用显示图标，根据容器大小适配
        {
            "src": "assets/imgs/48.png",
            "type": "image/png",
            "sizes": "48x48"
        },
        {
            "src": "assets/imgs/96.png",
            "type": "image/png",
            "sizes": "96x96"
        },
        {
            "src": "assets/imgs/192.png",
            "type": "image/png",
            "sizes": "192x192"
        }
    ],
    "background_color": "#2196F3", // 刚打开页面时的背景
    "theme_color": "#2196F3", // 主题颜色
    "display": "standalone", //独立显示
    "start_url": "index.html?launcher=true" // 启动的页面
}
```

