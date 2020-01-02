---
title: 前端性能优化—js代码打包
date: 2019-03-22 
categories: 
- web前端
tags:
- 性能优化
- 前端
- 打包
---

现在的 web 应用，内容一般都很丰富，站点需要加载的资源也特别多，尤其要加载很多 js 文件。js 文件从服务端获取，体积大小决定了传输的快慢；浏览器端拿到 js 文件之后，还需要经过解压缩、解析、编译、执行操作，所以，  **控制 js 代码的体积**  以及  **按需加载**  对前端性能以及用户体验是十分的重要。

本文从 Tree Shaking 和 代码分割 两部分介绍 js 打包优化，有兴趣的可以跟着一起实践。 clone 以下项目 [github.com/jasonintju/…](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fjasonintju%2Foptimizing-js) ，就是个简单的 React SPA，一看就懂。

## Tree Shaking

Tree Shaking 简单理解就是：打包时把一些没有用到的代码删除掉，保证打包后的代码体积最小化。其详细的介绍可以参考 [Tree-Shaking性能优化实践 – 原理篇](https://link.juejin.im/?target=https%3A%2F%2Fjuejin.im%2Fpost%2F5a4dc842518825698e7279a9) 。
<!--more-->
项目 clone、安装依赖后，先 npm run build 打包初始代码，大小及分布如下（其中 src/utils/utils.js 这个文件打包后大小为 11.72Kb ）：

src/containers/About/test.js 只引用但是没有使用到， src/utils/utils.js 这个文件是个 [工具](http://www.codercto.com/tool.html) 函数集，有很多很多函数，而我们只用到了其中的一个。默认情况下，整个文件都被打包进 main.js 了，显然，这是很大的冗余，正好可以使用 Tree Shaking 优化。

### 修改 .babelrc

{

  &quot;presets&quot;:[[&quot;env&quot;,{&quot;modules&quot;:false}],&quot;react&quot;,&quot;stage-0&quot;]

}

复制代码

### 修改 package.json

{

  &quot;name&quot;:&quot;optimizing-js&quot;,

  &quot;version&quot;:&quot;1.0.0&quot;,

  &quot;sideEffects&quot;:false

}

复制代码

这样设置之后，表示所有的 module 都是无副作用的，没有使用到的 module 都可以删掉，此时打包结果如下：

importReactfrom&#39;react&#39;;

// 只引入了 arraySum， utils.js 中的其他方法不会被打包

import{ arraySum }from&#39;@utils/utils&#39;;

import&#39;./test&#39;;// 引用，&quot;未使用&quot;，不会被打包

import&#39;./About.scss&#39;;// 引用，&quot;未使用&quot;，不会被打包

classAboutextendsReact.Component{

  render(){

    const sum = arraySum([12,3]);

    return(

      \&lt;div className=&quot;page-about&quot;\&gt;

        \&lt;h1\&gt;AboutPage\&lt;/h1\&gt;

        \&lt;div\&gt;12 plus 3 equals {sum}\&lt;/div\&gt;

      \&lt;/div\&gt;

    );

  }

}

exportdefaultAbout;

复制代码

如上面注释所说，Tree Shaking 认为这些是没有被使用的代码，所以可以删掉。但事实上我们知道不是这样的， test.js 可以删掉，但是 css、scss 是有用的代码，我们只需引入即可。因此，需要修改一下 sideEffects 的值：

{

  &quot;sideEffects&quot;:[

    &quot;\*.css&quot;,&quot;\*.scss&quot;,&quot;\*.sass&quot;

  ]

}

复制代码

表示，除了 [] 中的文件（类型），其他文件都是无副作用的，可以放心删掉。此时打包结果：

可以看到，css 等样式文件现在如期打包进去了。如果有其他类型的文件有副作用，但是也希望打包进去，在 sideEffects: [] 中添加即可，可以是具体的某个文件或者某种文件类型。

关于为什么修改这两个地方就可以实现 Tree Shaking 的效果了，可以参考一下 [developers.google.com/web/fundame…](https://link.juejin.im/?target=https%3A%2F%2Fdevelopers.google.com%2Fweb%2Ffundamentals%2Fperformance%2Foptimizing-javascript%2Ftree-shaking%2F) 或者其他文章，这里不做详细解释了。

## 代码分割

单页应用，如果所有的资源都打包在一个 js 里面，毫无疑问，体积会非常庞大，首屏加载会有很长时间白屏，用户体验极差。所以，要代码分割，分成一个一个小的 js，优化加载时间。

### 分离第三方库代码

第三方库代码单独提取出来，和业务代码分离，减少 js 文件体积。在 webpack.base.conf.js 中增加：

module:{...},

optimization:{

  splitChunks:{

    cacheGroups:{

      venders:{

        test:/node\_modules/,

        name:&#39;vendors&#39;,

        chunks:&#39;all&#39;

      }

    }

  }

},

plugins:...

复制代码

### 动态导入

使用 [ECMAScript 提案](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ftc39%2Fproposal-dynamic-import) 的 dynamic import 语法可以异步加载业务中的组件。使用方法如下：

// src/containers/App/App.js

// 注释掉此行代码

// import About from &#39;@containers/About/About&#39;;

// 修改模块为动态导入形式

\&lt;Route path=&quot;/about&quot; render={()=\&gt;import(/\* webpackChunkName: &quot;about&quot; \*/&#39;@containers/About/About&#39;).then(module=\&gt;module.default)}/\&gt;

复制代码

此时打包结果：

能看到， \&lt;About\&gt; 组件 已经被 webpack 单独打包出对应的 js 文件了。同时，结合 react-router ，分离 \&lt;About\&gt; 组件 的同时也做到了  **按需加载**  ：当访问 About 页面时， about.js 才会被浏览器加载。

注意，我们现在只是简单地使用了 dynamic import ，很多边界情况没考虑进去，比如：加载进度、加载失败、超时等处理。可以开发一个高阶组件，把这些异常处理都包含进去。社区有个很棒的 [react-loadable](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fjamiebuilds%2Freact-loadable) ，大树底下好乘凉~

npm i react-loadable

// src/containers/App/App.js

importLoadablefrom&#39;react-loadable&#39;;

// 代码分割 &amp; 异步加载

constLoadableAbout=Loadable({

  loader:()=\&gt;import(/\* webpackChunkName: &quot;about&quot; \*/&#39;@containers/About/About&#39;),

  loading(){

    return\&lt;div\&gt;Loading...\&lt;/div\&gt;;

  }

});

classAppextendsReact.Component{

  render(){

    return(

      \&lt;BrowserRouter\&gt;

        \&lt;div\&gt;

          \&lt;Header/\&gt;

          \&lt;Route exact path=&quot;/&quot; component={Home}/\&gt;

          \&lt;Route path=&quot;/docs&quot; component={Docs}/\&gt;

          \&lt;Route path=&quot;/about&quot; component={LoadableAbout}/\&gt;

        \&lt;/div\&gt;

      \&lt;/BrowserRouter\&gt;

    );

  }

}

复制代码

[react-loadable](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fjamiebuilds%2Freact-loadable%23loadablecomponentpreload) 还提供了 preload 功能。假如有统计数据显示，用户在进入首页之后大概率会进入 About 页面，那我们就在首页加载完成的时候去加载 about.js ，这样等用户跳到 About 页面的时候，js 资源都已经加载好了，用户体验会更好。

// src/containers/App/App.js

componentDidMount(){

  LoadableAbout.preload();

}

复制代码

如果有同学对Network面板不是很熟悉，可以看一下 [Chrome DevTools — Network](https://link.juejin.im/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000008407729) 。

### 提取复用的业务代码

第三方库代码已经单独提取出来了，但是业务代码中也会有一些复用的代码，典型的比如一些工具函数库 utils.js。现在， About 组件 和 Docs 组件 都引用了 utils.js ，webpack 只打包了一份 utils.js 在 main.js 里面，main.js 在首页就被加载了，其他页面有使用到 utils.js 自然可以正常引用到，符合我们的预期。但是目前我们只是把 About 页面异步加载了，如果把 Docs 页面也异步加载了会怎么样呢？

// src/containers/App/App.js

// 注释掉此行代码

// import Docs from &#39;@containers/Docs/Docs&#39;;

constLoadableDocs=Loadable({

  loader:()=\&gt;import(/\* webpackChunkName: &quot;docs&quot; \*/&#39;@containers/Docs/Docs&#39;),

  loading(){

    return\&lt;div\&gt;Loading...\&lt;/div\&gt;;

  }

});

classAppextendsReact.Component{

  render(){

    return(

      \&lt;BrowserRouter\&gt;

        \&lt;div\&gt;

          \&lt;Header/\&gt;

          \&lt;Route exact path=&quot;/&quot; component={Home}/\&gt;

          \&lt;Route path=&quot;/docs&quot; component={LoadableDocs}/\&gt;

          \&lt;Route path=&quot;/about&quot; component={LoadableAbout}/\&gt;

        \&lt;/div\&gt;

      \&lt;/BrowserRouter\&gt;

    );

  }

}

复制代码

此时打包结果：

能够看到，about.js 和 docs.js 里面都打包了 utils.js，重复了！ 在 webpack.base.conf.js 中增加：

module:{...},

optimization:{

  splitChunks:{

    cacheGroups:{

      venders:{

        test:/node\_modules/,

        name:&#39;vendors&#39;,

        chunks:&#39;all&#39;

      },

      default:{

        minSize:0,

        minChunks:2,

        reuseExistingChunk:true,

        name:&#39;utils&#39;

      }

    }

  }

},

plugins:...

复制代码

再打包看结果：

utils.js 也被单独打包出来了，达到了预期。

### 分离非首页使用且复用程度小的第三方库

假如，现在 Docs.js 引用了 lodash 这个三方库：

importReactfrom&#39;react&#39;;

import \_ from&#39;lodash&#39;;

import{ arraySum }from&#39;@utils/utils&#39;;

import&#39;./Docs.scss&#39;;

classDocsextendsReact.Component{

  render(){

    const sum = arraySum([1,3]);

    const b = \_.sum([1,3]);

    return(

      \&lt;div className=&quot;page-docs&quot;\&gt;

        \&lt;h1\&gt;DocsPage\&lt;/h1\&gt;

        \&lt;div\&gt;1 plus 3 equals {sum}\&lt;/div\&gt;

        \&lt;br /\&gt;

        \&lt;div\&gt;use \_.sum,1 plus 3 equals {b} too.\&lt;/div\&gt;

      \&lt;/div\&gt;

    );

  }

}

exportdefaultDocs;

复制代码

打包结果：

lodash.js 只在 Docs 页面使用，而且可能 Docs 页面访问量很少，把 lodash.js 打包在首页就会加载的 venders.js 里面，实在不是明智之举。

修改 webpack.base.conf.js ：

...

venders:{

  test:/node\_modules\/(?!(lodash)\/)/,// 去除 lodash，剩余的第三方库打成一个包，命名为 vendors-common

  name:&#39;vendors-common&#39;,

  chunks:&#39;all&#39;

},

lodash:{

  test:/node\_modules\/lodash\//,// lodash 库单独打包，并命名为 vender-lodash

  name:&#39;vender-lodash&#39;

},

default:{

  minSize:0,

  minChunks:2,

  reuseExistingChunk:true,

  name:&#39;utils&#39;

}

...

复制代码

此时把 lodash 单独打成了一个包，且配合 Docs 页面的按需加载，达到了理想的加载效果。

### 缓存

项目打包后，资源部署在 [服务器](http://www.codercto.com/category/server.html) 端，客户端需要向服务器请求下载这些资源，用户才能看到内容。使用缓存，客户端可以大大减少不必要的请求和时间耽搁，只有当资源有更新时，再去下载。区分一个文件是否有更新，使用 文件名 + hash 可以达到目的。本案例中，已经使用了 &#39;[name].[contenthash:8].js&#39; 。

然而，在打包的时候，webpack的运行时代码有时候会导致某些情况出现，如：什么内容都没改，两次 build 代码的 hash 不一样；或者是，修改了 a 文件的代码，却导致了某些未修改代码文件的 hash 也发生了变化。 [This is caused by the injection of the runtime and manifest which changes every build.](https://link.juejin.im/?target=https%3A%2F%2Fwebpack.js.org%2Fconcepts%2Fmanifest%2F)

注意：使用的 webpack 版本不同，可能会导致打包出的结果不一样。较新的版本或许没有这种 hash 问题，但为了安全起见，还是建议按照下面的步骤处理一下。

#### 分离 webpack runtimeChunk code

// webpack.base.conf.js

optimization:{

  runtimeChunk:{

    name:&#39;manifest&#39;

  },

  splitChunks:{...}

}

复制代码

此时，能达到：修改某个文件，只有这个文件和 manifest.js 文件的 hash 会发生变化，其他文件的 hash 不变。 打包前：

// About.scss

.page-about {

  padding-left:30px;

  color:#545880; // 修改字体颜色

}

复制代码

修改后：

#### HashedModuleIdsPlugin

增加、删除一些模块，可能会导致不相关文件的 hash 发生变化，这是因为 webpack 打包时，按照导入模块的顺序，module.id 自增，会导致某些模块的module.id 发生变化，进而导致文件的 hash 变化。

解决方式： 使用 webpack 内置的 [HashedModuleIdsPlugin](https://link.juejin.im/?target=https%3A%2F%2Fwebpack.js.org%2Fplugins%2Fhashed-module-ids-plugin%2F) ，该插件基于导入模块的相对路径生成相应的module.id，这样如果内容没有变化加上module.id 也没变化，则生成的 hash 也就不会变化了。

// webpack.prod.conf.js

const webpack =require(&#39;webpack&#39;);

...

plugins:[new webpack.HashedModuleIdsPlugin(),newBundleAnalyzerPlugin()]

复制代码

完整的优化代码见 [github.com/jasonintju/…](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fjasonintju%2Foptimizing-js-example)

有用的文章： [webpack分离第三方库及公用文件](https://link.juejin.im/?target=https%3A%2F%2Fyi-jy.com%2F2018%2F06%2F09%2Fwebpack-split-chunks%2F)

[developers.google.com/web/fundame…](https://link.juejin.im/?target=https%3A%2F%2Fdevelopers.google.com%2Fweb%2Ffundamentals%2Fperformance%2Foptimizing-javascript%2Ftree-shaking%2F)