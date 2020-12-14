---
layout: pages
title: webpack4打包css抽取
categories: 
- web前端
tags:
- webpack4
- css抽取
---
如下表示将所有 CSS 文件打包为一个（注意将权重设置为最高，不然可能其他的 cacheGroups 会提前打包一部分样式文件）   
```javascript
module.exports = {
  optimization: {
    splitChunks: {
      cacheGroups: {
        styles: {
          name: true,
          test: /\.css$/,
          chunks: 'all',
          enforce: true,
          maxSize: 1650000,//165kb，压缩拆包前的的最大值
          priority: 20, 
        }
      }
    }
  }
}
```