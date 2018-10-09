---
title: webpack caching
layout: post
---

# 如何利用 webpack 解决浏览器缓存问题
<div class="title-meta">
    <span><img class="title-category-img" src="../../../assets/images/categories/webpack.svg" alt="webpack"></span>
    <span><a class="github-link" href="/2018/09/24/webpack.html">Webpack</a></span>
    <span class="title-bullet">•</span>
    <span>Oct 09, 2018</span>
    <span class="title-bullet">•</span>
    <span>by <a class="github-link" href="http://github.com/limeii" title="http://github.com/limeii">Mei</a></span>
</div>

在文章 [浏览器缓存问题](/2018/09/21/issues-cache-busting.html) 有提到浏览器缓存的原理和利弊。


接下来就介绍怎么在webpack中解决浏览器缓存问题。

```
这篇文章是基于 webpack3.10.0

默认打包的时候是直接把CSS inline到bundle文件里面

```

### 缓存
webpack打包以后的文件一般都是：app.bundle.js/ vendor.bundle.js，每次发布以后，如果用户浏览器缓存没有过期，加上文件名字相同，浏览器不会从服务器下载最新文件，导致用户不能看到新发布的功能。


这就是我们所说的浏览器缓存问题。


解决方案就是，每次如果文件有改动，那么在发布的时候，就让对应bundle文件名不一样。这样，不管浏览器缓存过没过期，都会强制浏览器从服务器拿最新代码。


按照这个思路，一旦有文件改动，不管是 ts，js，img，css 等文件有改动，打包的时候，对应的bundle文件名也要有变化。

### chunkhash

在webpack中，build生成bundle文件的时候，在bundle文件名中加上hashcode，一旦有改动，那么bundle文件名中的hashcode也不一样，这样就保证了每次发布，bundle文件名不一样。

```
加上hashcode以后的bundle文件如下：

app.f68357fc8e99ece57afe.bundle.js

vendor.0d5cad4949ab26faaa39.bundle.js
```

具体用法是在webpack配置文件的output filename中加上 [chunkhash]：
```js
  //webpack.prod.js
  output: {
    path: path.join(process.cwd(), 'dist'),
    publicPath: "/dist/",
    filename: 'js/[name].[chunkhash].bundle.js',
  }
```
完整代码可参考：[angular-seed-project](https://github.com/LiMeii/angular-seed-project)


在terminal 跑命令行 npm run build:prod 之后生成文件如下：

![webpack-caching-chunkhash](https://limeii.github.io/assets/images/posts/webpack/webpack-caching-chunkhash.png){:height="100%" width="100%"}

```
里面有 0.068716ea07094f619891.bundle.js
1.3392ab62515e716550b2.bundle.js
2.f69ff655a0890c6a5b91.bundle.js
是因为用到angular lazy loading所产生的chunk文件
```

除了用 chunkhash 之外，还可以用 hash

### hash

把output filename中的chunkhash改为hash：
```js
  //webpack.prod.js
  output: {
    path: path.join(process.cwd(), 'dist'),
    publicPath: "/dist/",
    filename: 'js/[name].[hash].bundle.js',
  }
```
在terminal 跑命令行 npm run build:prod 之后生成文件如下：

![webpack-caching-hash](https://limeii.github.io/assets/images/posts/webpack/webpack-caching-hash.png){:height="100%" width="100%"}

从最终bundle文件名可以看出，所有bundle文件hashcode都是一样的。


我们来测试下：改动app.component.html文件，使用hash的方式，重新build以后，bundle文件如下：

```js
    0.64de26c53b0cd5ce3331.bundle.js   
    1.64de26c53b0cd5ce3331.bundle.js   
    2.64de26c53b0cd5ce3331.bundle.js   
    vendor.64de26c53b0cd5ce3331.bundle.js  
    app.64de26c53b0cd5ce3331.bundle.js  
    polyfills.64de26c53b0cd5ce3331.bundle.js  
```
所有的bundle文件名都改了，但hashcode都一样。



使用chunkhash方式，重新build以后，bundle文件如下：

```js
0.068716ea07094f619891.bundle.js
1.3392ab62515e716550b2.bundle.js
2.f69ff655a0890c6a5b91.bundle.js
vendor.0d5cad4949ab26faaa39.bundle.js 
app.c33736260b829570e8ba.bundle.js 
polyfills.9d98b7f13680b7f15ee4.bundle.js 
```
从最终bundle文件名可以看出，只有app.bundle文件的hashcode有改动，其他的都跟上一次contenthash方式编译结果一致。

**hash与chunkhash的区别：**


hash生成的bundle文件名中的hashcode是一样的，任一文件的改动，会导致所有bundle文件名都改变，每次发布以后强制客户端下载所有文件。


chunkhash生成的bundle文件名中的hashcode都不一样，文件改动，只会导致对应bundle文件名改变，发布以后，只会下载文件名改动的bundle文件，其他文件还是从缓存里拿。


**需要注意的是：hash 或 chunkhash最好只用在生产环境下，如果在开发环境下用，会导致编译变慢**


### 总结
这篇文章主要是介绍了，hash 和 chunkhash的基本用法，默认打包的时候是直接把CSS inline到bundle文件里面。


我们知道CSS在项目里面改动的次数不多，每次都inline到bundle文件里，会有以下问题：

- 如果本身CSS文件就很大，会导致bundle文件过大
- 任何一次非CSS代码改动，发布以后客户端会重新下载文件，虽然CSS没有变化，但是需要每次跟budnle文件一起下载


那么如何解决这个问题呢？


在[如何利用 webpack 优化打包 CSS 文件](/2018/10/10/webpack-css-extract.html) 文章中会结合 ExtractTextPlugin 和 CSS 介绍更全面的解决缓存问题方案。