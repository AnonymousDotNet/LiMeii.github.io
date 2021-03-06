---
title: Angular：性能优化清单
tags: Angular
layout: post
---

性能优化主要是从两方面入手，一个是网络性能，另一个是页面渲染。在具体介绍性能优化方式之前，先来解释下为什么网络性能和页面渲染会影响性能。

<blockquote>
<p>
在这里讨论的网络带宽的性能问题都是基于 HTTP/1.X，在 HTTP/2 中很多性能问题都解决了。
</p>
</blockquote>


在 HTTP/1.X 连接有三种方式：短链接，长连接和 HTTP 流水线。

![angular-performance](/assets/images/posts/angular/angular-performance01.png){:height="80%" width="80%"}

短链接是 HTTP/1.0 的默认模型，它每发一个请求时都会创建见一个新的 TCP 连接，收到 response 的时候就立马关闭连接，每次创建一个 TCP 连接都相当耗费资源，可想而知这种方式的性能很差，现在基本不用这种方式。


在 HTTP/1.1 以后就有了长连接和流水线，长连接是指创建一个 TCP 连接后，可以保持连接完成多次连续的请求，减少了打开 TCP 连接的次数，在 HTTP/1.1 以后的版本是默认的长连接的模式，长连接的缺点是，就算在空闲状态，它还是会消耗服务器资源。长连接是通过 ```Keep-Alive```消息头来控制。


默认情况下，HTTP 请求是按顺序发出的，下一个请求只有在当前请求收到应答以后才会发出，由于受到网络延迟和带宽的限制，在下一个请求发出可能需要等很长时间。流水线是指在同一个 TCP 长连接里连续发出请求，而不用等待前一个请求应答返回，理论上这种方式是最有效的，实现流水线是复杂的：传输中的资源大小，多少有效的 RTT 会被用到，还有有效带宽，流水线带来的改善有多大的影响范围。不知道这些的话，重要的消息可能被延迟到不重要的消息后面。这个重要性的概念甚至会演变为影响到页面布局！因此 HTTP 流水线在大多数情况下带来的改善并不明显，在 HTTP/2 已经有更好的方式替代流水线模式，流水线模式在浏览器里默认是不开启的。

基于 HTTP/1.X 的连接限制，我们在开发中过程要尽量减少网络带宽的占用，比如尽可能的减小文件的大小和减少请求次数，可以从以下几方面考虑：
- Bundling
- Code Splitting
- Minify，Uglify
- Compression
- Tree-shaking 
- Cache
- lazy loading 和 preloading
- 如果是用了前端框架，部署的时候用 AoT 的编译方式
- service worker


页面渲染，在正常情况下浏览器是60Hz的刷新率，每16.6ms会刷新一次页面，渲染页面的操作需要在这16.6ms内完成，否则就会导致页面失帧，那么提高页面渲染的性能，可以从以下几方面考虑：
- 减少页面重排和重绘，尽量避免以下会导致页面重排和重绘的操作：
   - 在 JS 代码中操作页面 DOM
   - 在页面 layout 稳定以后，增加和改变 CSS 样式
   - 改变：窗口大小，字体大小
   - 使用 table
   - 动画实现数度选择，动画速度越快，回流次数越多，也可以选择使用 requestAnimationFrame
   - 读取 offsetwidth offsetheight
   - CSS 伪类，比如 :hover 某个元素弹出一个消息框
- JS 全阻塞，CSS 半阻塞（会阻塞 JS 执行和渲染树，但不阻塞 DOM 构建）
   - JS 阻塞构建 DOM CSSOM 树，从而会阻塞构建渲染树，而且同时还会阻塞其他静态资源（图片）的下载，所以要把```<script>```标签放到body最后，或者是在标签里添加```defer``` 或者 ```async``` 属性。
   - CSS 文件下载解析，不会阻塞 HTML 文件解析，不会阻塞 DOM 树的构建，但是会阻塞 CSSOM 树，从而会阻塞渲染树的构建，所以 CSS 文件的连接可以放在 head 里，不影响
   - CSS 文件下载不会阻塞其他文件下载，但是会阻塞JS的文件执行
- web worker

## 网络性能

**推荐用 Angular 自带的编译方式打包**

Angular 内置了 webpack 打包方式，很多人在搭建 Angular(< 6.0) 项目的时候，发现 Angular 内置的 webpack 并不能满足实际项目打包的一些需求，所以通过```ng eject```把内置的 webpack.config 文件暴露出来，然后根据自己项目需求重写整个 webpack.config 配置，可以参考文章: 【[Angular：如何用Angular(<6.0)和Webpack搭建项目](https://limeii.github.io/2018/09/angular-webpack/)】


但是 Angular(>= 6.0)，去掉了```ng eject```命令，鼓励大家用 Angular 内置的打包方式开发 Angular 项目，因为```AoT``` ```Tree-Shaking``` ```ngc``` ```tsc``` ```minification``` ```Uglification``` ```Production mode```等等这些在 Angular 内置的打包方式中都是默认配置，不需要我们自己再去找第三方的 loader 或者 plugin 来实现类似的功能；而且从性能上说内置的打包方式要比我们自己搭的要好。


如何结合内置的打包方式和我们自己的 webpack 配置文件编译打包整个项目，可以参考文章：【[Angular：如何在Angular(8.0)中配置Webpack](https://limeii.github.io/2019/08/angular-customize-webpack/)】


**使用 AoT 编译**

在打包过程中使用 AoT 编译，指的是把所有 HTML component NgModule CSS 都编译成浏览器可以识别的 es5 代码。从而用户访问 Angular 应用的时候，只需要下载相应的 budnle 文件后直接渲染。不需要先下载```@angular/compiler```编译器编译模板文件，然后再渲染，大大提高了页面渲染性能。


在模板文件中 Angular 有自己的 directive component pipe，如果不先编译成 es6 代码，就没办法做 Tree Shaking。AoT(ngc) 编译，可以把模板文件先打包成 es6 (或者TypeScript)文件，这样就可以做 Tree Shaking，把一些没用的代码去掉，从而减少 bundle 文件的体积，也可以提高性能。


关于深入理解 Angular 编译，可以参考这篇文章：【[Angular：深入理解Angular编译机制](https://limeii.github.io/2019/08/angular-compiler/)】


**Tree Shaking**

如果项目中有一些代码(方法、文件)，在项目里完全没有被用到，这种代码称为 Dead Code。大量 Dead Code 如果编译打包进 bundle 文件，会导致 bundle 文件过大，页面渲染下载 bundle 文件的时候会浪费带宽，也会影响性能。


Tree Shaking 就是用来解决这种问题，它是指在编译打包过程中把 Dead Code 去掉，不把这些没用到的代码打包到最后的 bundle 文件里，从而可以减小 bundle 文件的体积，提高应用性能。可以把整个应用想象成一棵树，function/component/service/lib 好比是树叶，而那些定义但又没有被调用的 function/component/service/lib 好比是枯树叶，Tree Shaking 就是把那些枯树叶从树上摇下去。


关于更多 Tree Shaking 的理解和应用可以参考这篇文章： [Angular性能优化：Tree Shaking](https://limeii.github.io/2019/08/angular-tree-shaking/)


**lazy loading 和 preloading**

如果把一个大项目所有文件都打包进一个 bundle 文件，用户打开浏览器访问这个网站的时候，首先要从服务器下载这个超大的 bundle 文件，加上还需要时间做一些解析，会导致网站响应过慢。在 Angular 项目中可以通过配置 lazy loading 和 preloading 来提高性能。关于 lazy loading 和 preloading 在 Angular 项目中的理解和应用可以参考这篇文章：【[Angular：lazy loading和preloading](https://limeii.github.io/2018/09/angular-lazy-loading/)】


**webpack Code Splitting**

在项目中经常会有一些公用代码，被多个 module 或者 component 引用，如果在打包的时候直接把公用代码重复打包进不同的 bundle 文件，会造成代码冗余，也会影响应用性能。我们可以通过 webpack Code Splitting 做代码切割，把公用的代码单独提取出来放在 chunk 文件里，用户访问页面的时候只需要下载一次这个 chunk 文件就可以了。具体可以参考这篇文章：[webpack(3)：代码切割](https://limeii.github.io/2018/10/webpack-code-splitting/)


**使用 ChangeDetectionStrategy.OnPush 策略**

Angular 默认的变化检测机制是：异步事件 callback 结束后，NgZone 会触发整个组件树至上而下做变化检测，也就是说页面一个小小的 Click 事件就会触发所有组件的变化检测。虽然 Angular 变化检测本身性能已经很好了，在毫秒内可以做成百上千次变化检测。但是随着项目越来越大，其实很多不必要的变化检测还是会在一定程度上影响性能。在 Angular 中可以通过 OnPush 来跳过一些不必要的变化检测，从而优化整个应用的性能。更多关于 OnPush 策略的理解和应用可以参考这篇文章：【 [Angular Change Detection：变化检测策略](https://limeii.github.io/2019/06/angular-changeDetectionStrategy-OnPush/)】


**缓存**

- 用 RxJS 实现缓存效果
Angular 中通过 HttpClient 执行 Http Request 返回的 Observables 是 Cold Observable，这导致每次调用 API，都会生成一个新的 Observable 实例，有订阅之后才开始发送值，这也符合现在前端开发要求。但是实际开发过程中，有时候后端会有提供一些公用的常量 API，不同页面都需要用这些常量，按现在的调用 API 的方式，会导致常量 API 在不同的页面重复多次被调用，这种方式显然性能不好。可以通过 ReplaySubject 实现缓存效果，第一次调用常量 API 之后把这些常量缓存起来，之后调用同样的 API 就可以直接在 ReplaySubject 拿到值，不用每次都调用后端 API。具体实现可以参考文章：【[RxJS：如何通过RxJS实现缓存](https://limeii.github.io/2019/08/rxjs-caching/)】

- 选择合适的浏览器缓存策略
对于一些不经常改的静态资源，可以缓存在浏览器端，合理的缓存策略可以减少延迟，在重复利用缓存的资源文件同时，可以减少带宽和降低网络负荷，从而大大提高了性能。缓存机制可以参考文章：【[浏览器缓存机制：强缓存和协商缓存](https://limeii.github.io/2018/11/web-cache/)】

- 合理的利用浏览器数据存储
对于一些常用的数据，可以存在浏览器里，这样可以减少延迟和带宽，从而可以提高性能，浏览器数据存储方式有：Cookies、SessionStorage、LocalStorage、IndexedDB。对于这些存储方式的用法和区别可以参考文章：【[浏览器数据存储方式](https://limeii.github.io/2018/11/web-storage/)】

**防抖**

超高频触发网路请求，不仅效率低而且没办法保证请求结果的正确性，我们可以结合 RxJS 中的操作符```debounceTime``` ```map```  ```filter```  ```distinctUntilChanged``` 和```switchMap```实现防抖。具体可以参考文章：【[RxJS：如何用RxJS实现高效的HTTP请求](https://limeii.github.io/2019/08/rxjs-searchable-input/)】

**service worker**


**未完待续**
