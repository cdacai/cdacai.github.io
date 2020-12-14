---
title: webpack框架中es6语法及api的转换
date: 2020-01-07 22:09:35
categories: 
- web前端
tags:
- es6
- babel
- 浏览器兼容
---

低版本的浏览器语法转换安装以下几个插件并作配置如下： 
 安装babel包
``` 
`babel-core` `babel-loader` `babel-plugin-transform-runtime` `babel-polyfill``babel-preset-es2015`    
```
说明：   
1、babel-core、babel-loader、babel-preset-es2015这三个模块是必须的，安装最新的转换规则是这三个 @babel/core babel-loader @babel/preset-env
<!--more-->
2、babel有时转换语法并不会转化BOM里面不兼容的API比如 Promise,Set,Symbol,Array.from,async 等等的一些API，这时候就需要 polyfill 来转转化这些API。   
polyfill有三种：    
babel-runtime、 babel-plugin-transform-runtime、babel-polyfill。    
其中babel-runtime和 babel-plugin-transform-runtime的区别是，相当一前者是手动挡而后者是自动挡，每当要转译一个api时都要手动加上require('babel-runtime')，而babel-plugin-transform-runtime会由工具自动添加，主要的功能是为api提供沙箱的垫片方案，不会污染全局的api，因此适合用在第三方的开发产品中。

3、在转换 ES2015 语法为 ECMAScript 5 的语法时，babel 会需要一些辅助函数，例如 _extend。babel 默认会将这些辅助函数内联到每一个 js 文件里，这样文件多的时候，项目就会很大。所以 babel 提供了 transform-runtime 来将这些辅助函数“搬”到一个单独的模块 babel-runtime 中，这样做能减小项目文件的大小。    

transform-runtime优点:
* 不会污染全局变量   
* 多次使用只会打包一次   
* 依赖统一按需引入,无重复引入,无多余引入。
   
transform-runtime缺点:   
* 不支持实例化的方法Array.includes(x) 就不能转化
* 如果使用的API用的次数不是很多，那么transform-runtime 引入polyfill的包会比不是transform-runtime 时大     

4、网上很多人说，集成了transform-runtime就不用babel-polyfill了，其实不然，看看官方怎么说的：
  ```
  NOTE: Instance methods such as "foobar".includes("foo") will not work since that would require modification of existing built-ins (Use babel-polyfill for that).
  ```
 所以，我们还是需要babel-polyfill哦。

webpack.config.js 配置文件   
````
modules.exports = {
       ...
       entry: { name: ['babel-polyfill'], 'path/to/file.js' },  // 在入口文件添加babel-polyfill
       ....
       module: {
           rules: [
               {
                   test: /\.js$/,
                   exclude: /node_modules/,
                   loader: 'babel-loader'
               }  // 添加loader
           ]
       }
   }
````

.babelrc 配置文件 
 
 
 ```
{ 
  "presets": [ 
    "es2015" 
  ],
  "plugins": [
    "transform-runtime"
  ] 
}
```
   
