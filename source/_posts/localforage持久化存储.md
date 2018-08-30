---
title: localforage持久化存储
date: 2018-07-29 20:48:16
tags: [localforage, js, 前端存储, 技术]
reward: true
---

### 数据持久化

前端数据持久化是在前端业务加重的环境下所增加的一项很平常的需求。对于用户一些基本信息、token、以及一些应用中的配置项，存储在浏览器端，从而达到**记录状态**以及**离线共享**等功能。

<!--more-->

在之前的项目中，一直使用`localStorag`e 作为数据存储载体，其 API 简单方便，也能满足一般需求。但是，实际上，随着项目中对部分数据的存取频繁，则越来越发现`localStorage`的一些弊端：只能存取字符串类型的数据，对于对象的存取每次都要 JSON 序列化、反序列化。尤其是存取对象的时候，如果没有做统一的封装，则会引起多次序列化导致数据无法有效读取等问题。

### 更好的存储方式

最近在开发过程中，偶然发现调试界面还有除`localStorage`以外其他几个存储位:

![](https://ws1.sinaimg.cn/large/0064OUUqly1fpv411pd3fj308m09uglm.jpg)

然后对于 `WebSQL` 与 `IndenedDB`做了一些简单了解：

> Web SQL 数据库 API 并不是 HTML5 规范的一部分，但是它是一个独立的规范，引入了一组使用 SQL 操作客户端数据库的 APIs。

> IndexedDB 是一种低级 API，用于客户端存储大量结构化数据(包括, 文件/ blobs)。该 API 使用索引来实现对该数据的高性能搜索。虽然 Web Storage 对于存储较少量的数据很有用，但对于存储更大量的结构化数据来说，这种方法不太有用。IndexedDB 提供了一个解决方案。

从使用和定义上来看，`webSQL`更像是关系型数据库，`indexedDB`则更像 NoSQL。

### 兼容性

无论是`webSQL`还是`indexedDB`都是比`localStorage`更加合适的存储方案，但是前端开发，永远不能忽略的三个字就是：**兼容性**。

所以，又特意去查了一下二者的兼容性表：

![](https://ws1.sinaimg.cn/large/0064OUUqly1fpv4att22zj30zh0gzjsy.jpg)

![](https://ws1.sinaimg.cn/large/0064OUUqly1fpv7cnkddaj30zc0gp75v.jpg)

所以，目前来看，想要愉快的使用 indexedDB、webSQL 是不可能了。

然而，当我在**MDN**上查询`indexedDB`的时候，看到这样一句话：

> **注意**：IndexedDB API 是强大的，但对于简单的情况可能看起来太复杂。如果你更喜欢一个简单的 API，请尝试类库，如 localForage, dexie.js, 和 ZangoDB，使 IndexedDB 更方便用户。

于是，开始着手查阅 `localForage` 的文档:

> localForage is a JavaScript library that improves the offline experience of your web app by using an asynchronous data store with a simple, localStorage-like API. It allows developers to store many types of data instead of just strings.

> localForage includes a localStorage-backed fallback store for browsers with no IndexedDB or WebSQL support. Asynchronous storage is available in the current versions of all major browsers: Chrome, Firefox, IE, and Safari (including Safari Mobile).

> localForage offers a callback API as well as support for the ES6 Promises API, so you can use whichever you prefer.

然后，一发不可收拾。

根据 localForage 的文档描述，localForage 内部封装了对于本地数据库的各种存取接口，同时对于优先使用何种数据库也有可配置项，默认的是：`[localforage.INDEXEDDB, localforage.WEBSQL, localforage.LOCALSTORAGE]`，同时，localForage 支持结构化的数据存取，甚至是图片类型的数据。再者，它的 API 完全是基于 ES6 Promise 的异步 API。

![](https://ws1.sinaimg.cn/large/0064OUUqly1fpy22i3n9ij30f30feglz.jpg)

有了 localStorage 的最基本保证，我们可以放心的使用 localForage；

有了 localForage 的结构化数据存储，我们可以舒服的在前端存取各种各样的数据；

### 简单使用

- 配置

```JavaScript
localforage.config({
    name: '数据库名称',
    storeName: '表名或集合名',
    driver: '优先使用的存储方式（indexedDB || webSQL || localStorage）',
    size: 4980736,  // 数据库大小，只在使用webSQL时起作用
    version： 1.0,  // 数据库版本号
    description：'数据库描述'
});

// 注：name/storeName/version 都会改变数据库的基础属性。
```

- 存

```JavaScript
/ Unlike localStorage, you can store non-strings.
localforage.setItem('my array', [1, 2, 'three']).then(function(value) {
    // This will output `1`.
    console.log(value[0]);
}).catch(function(err) {
    // This code runs if there were any errors
    console.log(err);
});

// ES6 写法
localforage.setItem('my array', [1, 2, 'three'])
    .then(res => {
        console.log(res);
    }).catch(err => {
        console.log(err);
    });
```

- 取

```JavaScript
localforage.getItem('somekey').then(function(value) {
    // This code runs once the value has been loaded
    // from the offline store.
    console.log(value);
}).catch(function(err) {
    // This code runs if there were any errors
    console.log(err);
});

// ES6 写法
localforage.getItem('somekey'， (err, res) => {
    if (!err) {
        console.log(res);
    }
});
```

- 删

```JavaScript
localforage.removeItem('somekey').then(function() {
    // Run this code once the key has been removed.
    console.log('Key is cleared!');
}).catch(function(err) {
    // This code runs if there were any errors
    console.log(err);
});
```

- 清

```JavaScript
localforage.clear().then(function() {
    // Run this code once the database has been entirely deleted.
    console.log('Database is now empty.');
}).catch(function(err) {
    // This code runs if there were any errors
    console.log(err);
});
```

- 遍历

```JavaScript
// The same code, but using ES6 Promises.
localforage.iterate(function(value, key, iterationNumber) {
    // Resulting key/value pair -- this callback
    // will be executed for every item in the
    // database.
    console.log([key, value]);
}).then(function() {
    console.log('Iteration has completed');
}).catch(function(err) {
    // This code runs if there were any errors
    console.log(err);
});
```

### otherelse

虽然 localforage 为数据持久化以及跨浏览器兼容提供了很便利的方式，其 API 也相当简单，基于 Promise 的完全异步存取可以获得很高的 IO 效率，但是，对于一些需要频繁存取的字符串形的数据，则不必如此。完全异步的方式，反而在一些地方拖慢了效率。

Tips:

- 将这些数据使用 localStorage 来存取，在程序的任何地址只需要 localStorage[key]即可直接取到想要的值。
- 仍旧使用 localforage 存储，但是在程序初始化的时候，把后面频繁用到的数据，以全局变量或模块的方式暴露出去。

![image](https://ws1.sinaimg.cn/large/0064OUUqly1fnyhk1f22bj30zk0m8mxy.jpg)
