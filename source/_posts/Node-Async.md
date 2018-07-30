---
title: Node-Async
date: 2018-07-28 21:26:31
tags: [Node, Async, 技术]
---

# 安装依赖

```
npm install async
```

# 项目引入

```javascript
var async = require('async');
```

# 简介使用

<!--more-->

## series(tasks,callback);

series 函数，它的作用是串行执行，一个函数数组中的每个函数，每一个函数执行完成之后才能执行下一个函数，示例如下：

```javascript
var async = require('async');
async.series(
  [
    function(callback) {
      callback('test', 1);
    },
    function(callback) {
      callback(null, 2);
    }
  ],
  function(err, results) {
    console.log(results);
  }
);
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyxssqw2j30hf0270si.jpg)

## waterfall(tasks, [callback])

waterfall 和 series 函数有很多相似之处，都是按顺序依次执行一组函数，不同之处是 waterfall 每个函数产生的值，都将传给下一个函数，而 series 则没有这个功能，示例如下：

```javascript
var async = require('async');
async.waterfall(
  [
    function(callback) {
      callback(null, 1);
    },
    function(data, callback) {
      callback(null, 2);
    }
  ],
  function(err, results) {
    console.log(results);
  }
);
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyyrv8wuj308k0egdhq.jpg)

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyz2u9xzj308g0ebjt9.jpg)

## arallel(tasks, [callback])

parallel 函数是并行执行多个函数，每个函数都是立即执行，不需要等待其它函数先执行。

传给最终 callback 的数组中的数据按照 tasks 中声明的顺序，而不是执行完成的顺序，示例如下：

```javascript
var async = require('async');
async.parallel(
  [
    function(callback) {
      setTimeout(function() {
        callback(null, 'one');
      }, 1000);
    },
    function(callback) {
      setTimeout(function() {
        callback(null, 'two');
      }, 1000);
    }
  ],
  function(err, results) {
    console.log(results);
  }
);
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntz0asx9gj30cu02da9u.jpg)

## parallelLimit(tasks, limit, [callback])

parallelLimit 函数和 parallel 类似，但是它多了一个参数 limit。 limit 参数限制任务只能同时并发一定数量，而不是无限制并发，示例如下：

```javascript
var async = require('async');
async.parallelLimit(
  [
    function(callback) {
      setTimeout(function() {
        callback(null, 'one');
      }, 1000);
    },
    function(callback) {
      setTimeout(function() {
        callback(null, 'two');
      }, 1000);
    }
  ],
  1,
  function(err, results) {
    console.log(results);
  }
);
```

花费时间为 2 秒；

## whilst(test, fn, callback)

相当于 while，但其中的异步调用将在完成后才会进行下一次循环。当你需要循环异步的操作的时候，它可以帮助你。示例如下：

```javascript
var async = require('async');
var count = 0;
var list = [{ name: 'Jack', age: 20 }, { name: 'Lucy', age: 20 }];
async.whilst(
  function() {
    return count < 2;
  },
  function(callback) {
    list[count].age++;
    count++;
  },
  function(err) {
    console.log(list[0]);
    console.log(list[1]);
  }
);
```

## doWhilst(fn, test, callback)

相当于 do…while,较 whilst 而言，doWhilst 交换了 fn,test 的参数位置，先执行一次循环，再做 test 判断。

```javascript
var async = require('async');
var count = 0;
async.doWhilst(
  function(callback) {
    count++;
    setTimeout(callback, 1000);
  },
  function() {
    return count < 5;
  },
  function(err) {}
);
```

## until(test, fn, callback)

until 与 whilst 正好相反，当 test 条件函数返回值为 false 时继续循环，与 true 时跳出。其它特性一致。示例如下：

```javascript
var async = require('async');
var count = 5;
async.until(
  function() {
    return count < 0;
  },
  function(callback) {
    count--;
    setTimeout(callback, 100);
  },
  function(err) {
    console.log(count);
  }
);
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntz21i8wgj308702owea.jpg)

## doUntil(fn, test, callback)

doUntil 与 doWhilst 正好相反，当 test 为 false 时循环，与 true 时跳出。其它特性一致。

示例：

```javascript
var async = require('async');
var count = 5;
async.doUntil(
  function(callback) {
    count--;
    setTimeout(callback, 1000);
  },
  function() {
    return count < 0;
  },
  function(err) {}
);
```

## forever(fn, errback);

forever 函数比较特殊，它的功能是无论条件如何，函数都一直循环执行，只有出现程序执行的过程中出现错误时循环才会停止，callback 才会被调用。

示例：

```javascript
var async = require('async');
async.forever(
  function(next) {},
  function(err) {
    //
  }
);
```

## compose(fn1, fn2...);

使用**compose**可以创建一个异步函数的集合函数，将传入的多个异步函数包含在其中，当我们执行这个集合函数时，会依次执行每一个异步函数，每个函数会消费上一次函数的返回值。

我们可以使用**compose**把异步函数 f、g、h，组合成 f(g(h()))的形式，通过**callback**得到返回值，请看示例：

```javascript
var async = require('async');
function fn1(n, callback) {
  setTimeout(function() {
    callback(null, n + 1);
  }, 1000);
}
function fn2(n, callback) {
  setTimeout(function() {
    callback(null, n * 3);
  }, 1000);
}
var demo = async.compose(
  fn2,
  fn1
);
demo(4, function(err, result) {});
```

## auto(tasks, [callback]);

用来处理有依赖关系的多个任务的执行。示例如下：

```javascript
var async = require('async');
async.auto(
  {
    getData: function(callback) {
      callback(null, 'data', 'converted to array');
    },
    makeFolder: function(callback) {
      callback(null, 'folder');
    },
    writeFile: [
      'getData',
      'makeFolder',
      function(callback, results) {
        callback(null, 'filename');
      }
    ],
    emailLink: [
      'writeFile',
      function(callback, results) {
        callback(null, { file: results.writeFile, email: 'user@example.com' });
      }
    ]
  },
  function(err, results) {
    console.log('err = ', err);
    console.log('results = ', results);
  }
);
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntz360lg8j30cp03e3yf.jpg)

## queue(worker, concurrency);

queue 相当于一个加强版的 parallel，主要是限制了 worker 数量，不再一次性全部执行。当 worker 数量不够用时，新加入的任务将会排队等候，直到有新的 worker 可用。

它有多个点可供回调，如无等候任务时(empty)、全部执行完时(drain)等。

示例：定义一个 queue，其 worker 数量为 2,当 q 全部执行完时输出'work is over.'：

```javascript
var async = require('async');
var q = async.queue(function(task, callback) {
  console.log('worker is processing task: ', task.name);
  callback();
}, 2);
q.push({ name: 'foo' }, function(err) {
  console.log('finished processing foo');
});
q.push({ name: 'bar' }, function(err) {
  console.log('finished processing bar');
});
q.drain = function() {
  console.log('work is over.');
};
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntz3mqa07j30g704tmx6.jpg)

## apply(function, arguments..)

apply 是一个非常好用的函数，可以让我们给一个函数预绑定多个参数并生成一个可直接调用的新函数，简化代码。示例如下：

```javascript
var async = require('async');
var log = async.apply(console.log, '>');
log('good');
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntz3x252mj30eq03jt8j.jpg)

## iterator(tasks)

将一组函数包装成为一个 iterator，可通过 next()得到以下一个函数为起点的新的 iterator。该函数通常由 async 在内部使用，但如果需要时，也可在我们的代码中使用它。

```javascript
var iter = async.iterator([
  function() {
    console.log('111');
  },
  function() {
    console.log('222');
  },
  function() {
    console.log('333');
  }
]);
iter();
```

直接调用()，会执行当前函数，并返回一个由下个函数为起点的新的 iterator。调用 next()，不会执行当前函数，直接返回由下个函数为起点的新 iterator。

对于同一个 iterator，多次调用 next()，不会影响自己。如果只剩下一个元素，调用 next()会返回 null。
