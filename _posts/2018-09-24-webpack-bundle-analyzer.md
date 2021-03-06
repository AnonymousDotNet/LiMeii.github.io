---
title: webpack：分析和优化webpack bundle文件
tags: Webpack
layout: post
---


在用webpack打包的过程当中，曾经把webpack官方文档看了不下十遍，对于最终bundle文件里具体到底有什么或者某个特定的module到底被放到哪一个bundle文件里，始终都是一头雾水。
特别是代码切割这一块，看了那么多遍，并且google了各类大神对这一块的解释，还是一头雾水。直到我发现webpack-bundle-analyzer这个小插件，把它install到本地项目里跑起来，一下子就彻底明白了。


今天这篇文章就是给大家介绍webpack-bundle-analyzer。这个插件是build生成项目文件时同时生成一张所有bundle文件的动态treemap图片。
如下所示：


![treemap img]( https://limeii.github.io/assets/images/posts/webpack/webpack-bundle-analyzer.gif){:height="100%" width="100%"}


在这个树形图片里，会有包含下面的内容：


1. 每个打包以后的bundle文件里面，真正包含哪些内容，项目里的module、js、component、html、css、img最后都被放到哪个对应的bunlde文件里了。

2. 每个bundle文件里，列出了每一个的module、componet、js具体size，同时会列出start size、parsed size、gzip size这三种不同的形式下到底多大，方便优化。

<blockquote>
<p>start size：原始没有经过minify处理的文件大小</p>
<p>parse size：比如webpack plugin里用了uglify，就是minified以后的文件大小</p>
<p>gzip size：被压缩以后的文件大小</p>
</blockquote>


基于以上给出的信息，
你就能比较直观的在图片里看到，哪些公用library被重复打包到不同的bundle文件里，或者是说哪一个过大影响性等等；从而你就可以对你的webpack打包方式进行优化。

## 用法

 先在项目里面安装这个plugin:

```js
npm install --save-dev webpack-bundle-analyzer
```

然后在webpack config文件里面加上以下代码:

 ```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = { 
  plugins: [

     new BundleAnalyzerPlugin()
  ]
};
 ```


 最后run webpack build命令，比如```npm run build```，在build结束以后，默认会直接在浏览器里把最终的动态treemap图片展示出来


 关于更多的用法，可以参照官方文档 [webpack bundle analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)