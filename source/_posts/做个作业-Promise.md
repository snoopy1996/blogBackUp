---
title: 做个作业_Promise
date: 2018-10-28 12:12:09
tags: [技术, Promise, 实现]
---

之前就一直想专门写一篇自实现`Promsie`的文章，但是由于各种事情，耽搁了下了，一直在代办事项列表里。

事实证明，有些事情你有计划，但是没做，是要摔跤的。

> 来吧，实现一个 Promsie(一点资讯技术面)

<!--more-->

**注**：本篇共分为两部分，一是凭借理解中的 Promise 相关特性的实现（异步控制+链式调用+捕获错误，中断链式调用），二是根据正经的`Promise / A+ 规范`来一点点测试实现。

## 个人理解版

### 梳理思路

首先，要看一下平时是怎么使用`Promise`的，然后把所需要的属性方法罗列出来。

```javaScript
new Promise((resolve, reject) => {
    resolve(1);
})    // 构造函数传入fn
    .then(res => {}, err => {})    // then回调传入结果
    .then(res => {}, err => {})    // then回调传入结果
    .catch(err => {});  // 错误捕获
```

由上可知，`Promsie`本身有四个外露的属性方法:

- `resolve` 负责修改状态为成功态（fulfilled）
- `reject` 负责修改状态为失败态（rejected）
- `then` 负责上一次之后的结果处理，并返回一个新的 Promise 实例或原有的 Promsie 实例（实际上是新的 Promise 实例，规范规定除了状态可由`pending`转换一次以外，其他都不能改变）
- `catch` 负责失败之后的原因处理

先简要定义出来：

```javaScript
// 定义三种状态
const successStatus = 'FULLFILLED';
const failStatus = 'REJECTED';
const waitStatus = 'PENDING';
function myPromise(fn) {
    this.status = waitStatus;   // 初始化Promise状态
    this.value = null;
    this.err = null;
    // 四个方法
    this.resolve = resolve;
    this.reject = reject;
    this.then = then;
    this.catch = catchErr;

    // 执行回调
    fn(this.resolve, this.reject);
}

function resolve(res) {};
function reject(err) {};
function then(fn1, fn2) {};
function catchErr(fn) {};
```

然后，依次处理每个函数。

首先是 resolve， 根据 resolve 函数的作用：修改状态，然后传入成功的结果值。（reject 也一样）

```javaScript
function resolve(res) {
    if (this.status === waitStatus) {
        this.status = successStatus;
        this.value = res;
    }
};
function reject(err) {
    if (this.status === waitStatus) {
        this.status = failStatus;
        this.err = new Error(err);
    }
};
```

然后是 `then`，这个方法明显是需要 resolve 之后执行，并且链式调用的根源也在 then 这里，因为他返回了一个新的`Promise`实例

```javaScript
function then(successCallback, errorCallback) {
    if (this.status === successStatus) {
        return new Promise(resolve => resolve(successCallback(this.value)));
    }
    if (this.status === failStatus) {
        return new Promise((resolve, reject) => reject( errorCallback(this.err)));
    }
}

function catchErr(errorCallback) {
    return new Promise((resolve, reject) => reject( errorCallback(this.err)));
}
```

### 简单实现

看起来，好像可以了，emmm , TRY

```javaScript
const successStatus = 'FULLFILLED';
const failStatus = 'REJECTED';
const waitStatus = 'PENDING';
function MyPromise(fn) {
  const _this = this;
  _this.status = waitStatus; // 初始化Promise状态
  _this.value = null;
  _this.err = null;

  // 四个方法
  _this.resolve = function(res) {
    if (_this.status === waitStatus) {
      _this.status = successStatus;
      _this.value = res;
    }
  };
  _this.reject = function(err) {
    if (_this.status === waitStatus) {
      _this.status = failStatus;
      _this.err = err;
    }
  };
  _this.then = function(successCallback, errorCallback) {
    if (successCallback && _this.status === successStatus) {
      return new Promise(resolve => resolve(successCallback(_this.value)));
    }
    if (errorCallback && _this.status === failStatus) {
      return new Promise((resolve, reject) => reject(errorCallback(_this.err)));
    }
    return new Promise((resolve, reject) =>
      reject('then is start, but status no change')
    );
  };
  _this.catch = function(errorCallback) {
    return new Promise((resolve, reject) => reject(errorCallback(_this.err)));
  };

  // 执行回调
  fn(_this.resolve, _this.reject);

  // console.log('_this', _this);
}

new MyPromise((resolve, reject) => resolve(1))
  .then(res => {
    console.log('res->', res);
    return res + 1;
  })
  .then(res => {
    console.log('res->', res);
    return res + 1;
  })
  .then(res => {
    console.log('res->', res);
    return res + 1;
  })
  .catch(err => {
    console.log('err', err);
  });

new MyPromise((resolve, reject) => reject(1))
  .then(res => {
    console.log('res->', res);
    return res + 1;
  })
  .then(res => {
    console.log('res->', res);
    return res + 1;
  })
  .then(res => {
    console.log('res->', res);
    return res + 1;
  })
  .catch(err => {
    console.log('err', err);
  });
```

结果 ->

![result](https://ws1.sinaimg.cn/large/0064OUUqly1fwnqot85qnj30az02p3yb.jpg)

从结果来看，似乎是成功了，然而，加上异步呢？

```javaScript
new MyPromise(resolve => {
  setTimeout(function() {
    console.log('this is first resolve');
    resolve(1);
  }, 2000);
})
  .then(res => {
    console.log('res->', res);
    return new Promise(resolve => {
      setTimeout(function() {
        resolve(res + 1);
      }, 1000);
    });
  })
  .catch(err => {
    console.log('err', err);
  });
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fwnqw8omtzj30ea01t0sn.jpg)

好吧，明显出问题了。

简单分析一下，可以发现，是由于在 `resolve`执行之前，`then`的链式调用就已经开始执行了，导致执行`then`的时候，状态还是`pending`，所以就报错了~~~

### 异步处理

Promise 所解决的是异步回调的问题，链式调用相比于异步回调嵌套要清晰很多，所以，称之为“优雅”。但问题是，我们如何把一个嵌套调用改成看起来是链式调用的形式呢？

```javaScript
function a(b) {
    b(c);
};
function b(c) {
    c();
}
function c() {}
a(b(c));
```

所以，要想办法保证 a、b、c 依次执行，那么顺序的数据结构，就试试数组。
利用 `[f1, f2, f3]`来执行下去。如果没法保证 then 的执行顺序，那么就需要保证所有的 then 的回调的执行顺序。这样一来，目标就清晰很多。

可以先用一个数组保存所有的成功回调函数，然后再依次执行下去。每次将上一个回调函数的结果保留传入下一次回调。那么，then 函数就成了一个收集器的作用。

```javaScript
const successStatus = 'FULLFILLED';
const failStatus = 'REJECTED';
const waitStatus = 'PENDING';
function MyPromise(fn) {
  let _this = this;
  _this.status = waitStatus; // 初始化Promise状态
  _this.value = null;
  _this.err = null;
  _this.successCallbacks = [];
  _this.errorCallbacks = [];
  // 四个方法
  _this.resolve = function(res) {
    if (_this.status === waitStatus) {
      _this.status = successStatus;
      _this.value = res;
      setTimeout(function() {
        console.log('执行fn', _this.successCallbacks.length);
        // 保证执行时候，then已经执行完
        if (_this.successCallbacks.length) {
          let fn = _this.successCallbacks.shift();
          _this.value = fn(_this.value);
          while (_this.value instanceof MyPromise !== true) {
            let fn = _this.successCallbacks.shift();
            _this.value = fn(_this.value);
          }
          if (_this.value instanceof MyPromise) {
            console.log('执行MyPromise------');
            let successCallbacks = _this.successCallbacks;
            _this = _this.value;
            _this.successCallbacks = successCallbacks;
          }
        }
      }, 0);
    }
  };
  _this.reject = function(err) {
    if (_this.status === waitStatus) {
      _this.status = failStatus;
      _this.err = err;
      setTimeout(function() {
        console.log('执行fn', _this.errorCallbacks.length);
        // 保证执行时候，then已经执行完
        let fn = _this.errorCallbacks.pop();
        _this.err = fn(_this.err);
      }, 0);
    }
  };
  _this.then = function(successCallback, errorCallback) {
    if (successCallback) {
      _this.successCallbacks.push(successCallback);
    }
    if (errorCallback) {
      _this.errorCallbacks.push(errorCallback);
    }
    return _this;
  };
  _this.catch = function(errorCallback) {
    _this.errorCallbacks.push(errorCallback);
    return _this;
  };
  // 执行回调
  fn(_this.resolve, _this.reject);
}

new MyPromise(resolve => {
  setTimeout(function() {
    console.log('this is first resolve');
    resolve(1);
  }, 2000);
})
  .then(res => {
    console.log(res);
    return res + 1;
  })
  .then(res => {
    console.log(res);
    return res + 1;
  })
  .then(res => {
    console.log(res);
    return res + 1;
  })
  .then(res => {
    console.log('res1->', res);
    return new MyPromise(resolve => {
      setTimeout(function() {
        resolve(res + 1);
      }, 1000);
    });
  })
  .then(res => {
    console.log('res2->', res);
    return new MyPromise(resolve => {
      setTimeout(function() {
        resolve(res + 1);
      }, 1000);
    });
  })
  .catch(err => {
    console.log('err', err);
  });
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fwnt5oluu1j30e406fwem.jpg)

当然，以上代码已经完全没有规范定义的那些内容，只有 异步控制+链式调用。

## Promise/A+ 规范版本

[Promise/A+ 规范译文](http://www.ituring.com.cn/article/66566)

路还很长~~

下次补全
