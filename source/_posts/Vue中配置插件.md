---
title: Vue中配置插件
date: 2018-07-28 21:10:07
tags: [Vue, 插件配置, 技术]
---

# Vue-cli 项目配置插件

JQuery 为例

## NPM 安装库

```
npm install jquery --save
```

<!--more-->

## webpack resolve 修改

```javascript
resolve: {
      extensions: ['', '.js', '.vue'],
      fallback: [path.join(__dirname, '../node_modules')],
      alias: {
          'src': path.resolve(__dirname, '../src'),
          'assets': path.resolve(__dirname, '../src/assets'),
          'components': path.resolve(__dirname, '../src/components'),

          // webpack 使用 jQuery，如果是自行下载的
          // 'jquery': path.resolve(__dirname, '../src/assets/libs/jquery/jquery.min'),
          // 如果使用NPM安装的jQuery
          'jquery': 'jquery'
      }
   },
```

## webpack plugins 修改

```javascript
plugins: [
      new webpack.ProvidePlugin({
          $: "jquery",
          jQuery: "jquery"
      })
   ],
```
