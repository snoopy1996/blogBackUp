---
title: Vue初始化项目时已有history影响返回
date: 2018-07-28 21:16:09
tags: [Vue, history, 技术]
---

# Vue 初始化项目时已有 history 影响返回

有时候由于第三方限制，单页应用需要先跳出应用然后经过某些操作，再跳转回来。但是由于这时候当前窗口已产生历史记录，而且浏览器又没有提供删除 history 的 api，所以造成用户点击返回影响使用效果。

<!--more-->

所以，我们可以在初始化应用之前保存一下当前`history.length`，然后在全局返回事件那里做一下判断，如果当前`history.length` > 本地保存的初始化时历史长度，则允许返回，否则特定 replace 跳回主页，或根据判断 replace 跳回某个页面。

![](https://ws1.sinaimg.cn/large/0064OUUqly1fpp74d9utcj30re08e754.jpg)

![](https://ws1.sinaimg.cn/large/0064OUUqly1fpp75f1pwej30lz052t91.jpg)
