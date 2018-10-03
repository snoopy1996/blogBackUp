---
title: 经典闭包问题与Promise
date: 2018-09-12 18:14:36
tags: [技术,闭包,Promise]
---
### what

```javasCript
for(var i = 0; i < 10; i++) {
    setTimeout(function() {
        console.log(i);
    }, 500);
}
// 依次打印出 10 10 10 10 10 10 10 10 10 10
```
<!--more-->

### why

这是JS中非常经典的问题，常用来考察`闭包`的概念。

出现这个结果的原因其实很简单，由于JS是**单线程且异步**的，当遇到耗时操作的时候，他**不会等待**耗时操作执行完就会直接继续执行后续的操作，而是把可能会阻塞代码执行的放到一边去执行。

在这段代码里，`setTimeout`明显就是那段会阻塞代码执行的存在，所以，被V8扔到了一边，而当所有的循环执行结束之后，等到执行`console.log(i)`的时候，`i`已经被赋值成了10，并且，这十个延时器计时几乎是同时开始，也会同时结束，最终**同时打印**出10个10。

P.S: 如果把`500`改成`0`，结果仍然是10个10。在浏览器中的表面原因就是，浏览器在实现setTimeout函数的时候，最极限值是`16ms`（其实是15.6ms, 就是常见的是64fps，另外， 现代浏览器chrome/ie10已经将极限值提升了大概4倍左右，也就是 4ms），所以你即使写成了0，但是在具体实现的时候还是16ms。

可以适当了解一下`requestAnimationFrame`与`setTimeout`的代替解决方案`setImmediate`

### what should we do?

既然问题是在执行`console.log(i)`的时候，`i`的值被改变了，没有达到我们的预期，那么就需要一个方法来保留`i`的值——闭包。

> 函数对象可以通过作用域链相互关联起来，函数体内的变量都可以保存在函数作用域内，这种特性在计算机科学文献中称为“闭包”。(函数将变量“包裹”起来)

从技术的角度来讲，所有的javaScript函数都是闭包，它们都是对象，它们都关联到作用域链。但是，只有当某个函数内部声明了一个新函数，且在新函数内部访问了这个函数的局部变量的时候，局部变量的值被保存到作用域中，才算是真正用到了闭包的特性。

所以，可是尝试用闭包，解决上述问题：

```javaScript
for (var i = 0; i < 10; i++) {
  (function(j) {
    setTimeout(function() {
      console.log(j);
    }, 500);
  })(i);
}
// 依次打印出 0 ~ 9
```

### otherelse

#### 1. setTimeout的第三个参数

继续看setTimeout的调用，在callback函数里，`i`实际上是变量，而非常量，而这个常量的来源就是for循环时通过var声明的`i`。所以，在实际执行`console.log`的时候，打印的是 `i`， 而不是 0,1,2,3……

既然如此，如果我们把来源改一下，那么也能解决这个问题。

很明显，可以这样：

```javaScript
for (var i = 0; i < 10; i++) {
  var a = i;    // a的值被保存，也是一种闭包的使用
  setTimeout(function() {
    console.log(a);
  }, 0);
}
// 打印 0 ~ 9
```

##### 仔细看看callback

setTimeout是高阶函数，高阶函数的特点就是函数作为参数，而函数本身则是允许我们去传参的，如果我们能把`i`的值作为参数传入callback，理论上也可以解决这个问题。可是，setTimeout只允许了我们声明callback，而调用则是在其内部封装，所以，该怎么传参呢？

其实很多人在用setTimeout的时候都是`setTimeout(callback, milliseconds);`这样来使用，而不知道，实际上setTimeout还有第三个参数，或者说还有第3~n个参数，在IE9及以上的浏览器中，支持将这些参数传给callback，供其使用。

所以，也可以这样来解决：

```javaScript
for (var i = 0; i < 10; i++) {
  setTimeout(function(j) {
    console.log(j);
  }, 0, i);
}
// 打印 0 ~ 9
```

#### 2. 把焦点关注在var上怎么解决

前面说过，callback中的`i`是变量，而不是常量，而在打印的时候，i是唯一的，就是循环后已经被赋值为10的变量。如果在for循环的后面紧接着执行 `i = 100;` 那么打印出来的就会是十个100。

如果，我们把 var 声明改成 let 声明会怎么样？

因为JS原本只有全局作用域和函数作用域，而ES6引入的let实际是为JS引入了块级作用域。很明显，`for(){}`明显是块级作用域，所以，如果我们使用`let`代替`var`,因为`let`的赋值只在其块级作用域内生效，所以理论上也是可以解决问题的。

```javaScript
for (let i = 0; i < 10; i++) {
  setTimeout(function() {
    console.log(i);
  }, 0);
}
// 打印 0 ~ 9
```

### 用Promise代替setTimeout会怎么样？

众所周知，Promise的出现就是为了解决JS中的异步回调地狱问题，其链式引用相比于回调的写法，不知优雅了多少倍：

```javaScript
function A(callBack){   callBack(); }
function B(callBack){   callBack(); }
function C(callBack){   callBack(); }
function D(callBack){   callBack(); }

D(C(B(A))); // Oh!  Shit!!!
```

```javaScript
Promise.resolve(1)
    .then(res => {})
    .then(res => {})
    .then(res => {})
    .then(res => {})
    .cache();   // profect!!!
```

Promise是异步回调的解决方案，那么，如果在这里使用Promise代替SetTimeout会发生什么奇妙的化学反应？

```javaSciprt
for (var i = 0; i < 10; i++) {
  Promsie.resolve(i).then(res => {
    console.log(res);
  });
}
```

Q：如果这样写什么结果？

A：应该是 0 ~ 9吧。

Q：Promise不也是异步的么？for循环遇到它不应该放到后面执行吗？为什么这里和SetTimeout的结果不一样了？

**猝！**

------------

好吧，开始探索一下Promise.

探索之前，当然是要执行一下代码看看结果：

![打印出0~9](https://ws1.sinaimg.cn/large/0064OUUqly1fv6vvscev4j309a05l744.jpg)；

首先，`Promise`的`then`和`catch`都是返回一个新的`Promise`对象，所以，才能多次连续链式调用。当然，这和结果为什么是0~9没啥关系。

看代码很明显能看到，for循环内的Promise一共调用了两个函数，一个是 `resolve` 一个是 `then`。

思考：如果Promise直接是异步的，那么整个`Promise()`应该都会在`for`循环之后调用，这样的话，那么和`setTimeout`的回调结果应该是一样的十个10才对，而不是0~9。所以，肯定不是这样的。

接下来，改造一下代码，看看执行顺序：

```javaScript
for (var i = 0; i < 10; i++) {
  console.log('循环内, i->', i);
  new Promise(resolve => {
    console.log('resolve内 ->');
    resolve(i);
    console.log('resolve 执行之后');
  }).then(function(res) {
    console.log(res);
  });
}
```

![输出结果](https://ws1.sinaimg.cn/large/0064OUUqly1fv6wdy3knoj30b9064mxd.jpg)

从打印结果来看， Promise的构造函数就是一个普通的函数，会在`for`循环内立即执行，而`resolve()`函数，就显得比较特殊了。

同时，这样来看，`resolve(i)`的时候，把`i`作为参数传入，所以`i`的值可以被保留下来，然后在`then()`里接收并打印出来。

-----

另附**Promise基本状态说明：**

- Promise存在三个状态（state）pending、fulfilled、rejected
- pending（等待态）为初始态，并可以转化为fulfilled（成功态）和rejected（失败态）
- 成功时，不可转为其他状态，且必须有一个不可改变的值（value）
- 失败时，不可转为其他状态，且必须有一个不可改变的原因 (reason)
- `new Promise((resolve, reject)=>{resolve(value)})` resolve为成功，接收参数value，状态改变为fulfilled，不可再次改变。
- `new Promise((resolve, reject)=>{reject(reason)})` reject为失败，接收参数reason，状态改变为rejected，不可再次改变
- 若是executor函数报错 直接执行reject();

另外，`then`函数接收两个函数：`onFulfilled`和`onRejected`，`onFulfilled`是执行成功，`onRejected`是执行失败，可以捕获`rejected`的错误信息。同时，`Promise`的链式调用还有一个`catch`函数，此函数也用于捕获错误，与`onRejected`相比较，`catch`不仅能捕获`Promise`回调函数里的错误，还能捕获`onFulfilled`函数里的错误，同时由于事件的冒泡机制，最后会被最后一个`catch`捕获。