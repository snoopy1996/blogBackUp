---
title: JS闭包与内存泄漏
date: 2018-10-12 12:03:57
tags: [技术,闭包,内存泄漏,GC]
---
> 函数对象可以通过作用域链相互关联起来，函数体内部变量都可以保存在函数作用域内，这种特性被称为——闭包。

从根本的定义来说，所有的JavaScript函数都是闭包，他们都是对象，且都关联到作用域链。

但是，常常在面试中被问到的闭包，大多都是指的特殊情况下，即“内部函数访问了外部作用域的变量，导致该变量一直在内存中，可以一直访问”的情况。
<!--more-->
#### 内存泄漏的定义
紧接着，就是**闭包的危害**，属于老生常谈的内容了，引用变量如果一直存在，那么势必会造成内存泄漏。

所以，到底什么是内存泄漏？

> 内存泄漏指由于疏忽或错误造成程序未能释放已经不再使用的内存。内存泄漏并非指内存在物理上的消失，而是应用程序分配某段内存后，由于设计错误，导致在释放该段内存之前就失去了对该段内存的控制，从而造成了内存的浪费。(维基百科)

所以，更严谨的来说，内存泄漏并没有什么危害，有危害的是**不被释放的变量的堆积导致的大量的内存泄漏引起内存不足，程序运行缓慢甚至于崩溃**。
#### 如何回收
在这里，要很感谢“OYO的前端面试官”的提问：怎样才能回收闭包中泄漏的内存呢？

一般来说，如果需要手动回收引用变量内存，最简单的方法就是置空，即`variableName = null;`，但问题是，我们提到的闭包是因为我们无法直接访问该变量，所以才会利用闭包来保存它并控制它，所以，我们的目的是**不希望它被回收**，所以才使用闭包。从这个角度来说，的确几乎从未考虑过手动回收的问题。

后来，我仔细想了想，如果真的要回收内存的话，可能我个人能想到的，只有一些特殊的写法实现：
```javaScript
/*supersy*/
function checkScope() {
  var temp = {desc: '我不想被GC'};
  return function(gc) {
      if (gc) {
          temp = null;
      }
      return temp;
  }
}
var getTemp = checkScope();
// 正常使用
getTemp();
// 回收
getTemp(0);
```
#### 测试一下到底什么情况下会导致不被GC回收
接下来，还是仔细看一下，闭包对内存泄漏的影响到底有多大：

先写个测试页面

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Closure</title>
</head>

<body>
  <p>不断单击【click】按钮</p>
  <button id="click_button">Click</button>
  <script>
    // 本函数只用于测试使用，或者说是为了测试而写
    // 实际开发中，不会有人在函数内声明一个不会被调用的函数来访问变量形成闭包的
    function f() {
      var str = Array(10000).join('#');
      var foo = {
        name: 'foo'
      }
      function unused() {
        var message = 'it is only a test message';
        str = 'unused: ' + str;
      }
      function getData() {
        return 'data';
      }
      return getData;
    }

    var list = [];
    
    document.querySelector('#click_button').addEventListener('click', function () {
      list.push(f());
    }, false);
  </script>
</body>

</html>
```

在Chrome浏览器总打开，F12调试工具选中**Memery**

![Memery示意图](https://ws1.sinaimg.cn/large/0064OUUqly1fw581mrtpij30ow0djdh2.jpg)

1. 选中 timeLine ，记录时间节点时的内存使用问题
2. 手动执行一下 GC
3. 点击开始记录按钮
4. 间隔性的点击页面上的 click 按钮
5. 再次点击记录按钮，停止记录

大概就是这样一种结果，每个**蓝色竖条**都是随着时间新分配的内存。
![记录结果](https://ws1.sinaimg.cn/large/0064OUUqly1fw584ip1g2j30g50f3gm7.jpg)

鼠标移上去，选中其中一项，然后逐个分析。

![内存使用情况查看](https://ws1.sinaimg.cn/large/0064OUUqly1fw58zldleej30ns0kkgn3.jpg)

图中可以明显看到，函数作用域中没有被调用使用的变量`str`也被保存在作用域中。按理来说，返回的是`getData`，并不是`unused`，为什么会没有被GC回收呢？ 

所以，如果去除掉对`str`的引用，是否就可以解决这个问题？

```javaScript
function f() {
    var str = Array(10000).join('#');
    var foo = {
        name: 'foo'
    }
    function unused() {
        var message = 'it is only a test message';
        // str = 'unused: ' + str;  // 注释
    }
    function getData() {
        return 'data';
    }
    return getData;
}
```
![二次结果](https://ws1.sinaimg.cn/large/0064OUUqly1fw58xnli3ej30cv0cqgmf.jpg)

可以明显看到不仅str被回收，而且申请的内存空间大小也明显减小，从10KB -> 232B。

原来，相同作用域内创建的多个内部函数是共享同一个**变量对象**的，也就是说，如果所有的内部函数都没有被return调用，那么整个变量对象会被GC，但是只要其中某一个被使用，那么整个变量对象就都会被保留，因而上述代码保留了`str`。

所以，目前阶段问题的分析结果是：

*引入闭包函数，且在其内部函数中有无用的引用，则此段引用被保存在内存中，无法被释放，导致内存泄漏。*

另外，可以看到的是，所谓闭包，实际上是返回了一个对象。如果我们真的需要在闭包之后，手动释放内存空间，为什么不用构造函数呢？

```javaScript
function CheckScope() {
    this.temp = {desc: '不想被GC'};
    this.getTemp = funtion() {
        return this.temp;
    }
}
var checkScope = new CheckScope();
// 使用
checkScope.getTemp();
// 回收
checkScope = null;
```

#### JavaScript与GC

研究完闭包与内存泄漏的关系，接下来就要好好看看 GC机制 了。

查过一些资料与一些技术社区的相关问答

> 在低版本ie下dom是com，它采用的垃圾回收机制是引用计数，如果在闭包中引用了dom那么它的引用计数会一直大于0，会一直无法回收，这就是闭包导致内存泄露的特殊情况。

##### 如何判断是否可以GC?

关于如何判断是否可以GC，主要有两种方式：

- 标记清除：
    js中最常用的垃圾回收方式就是标记清除。当变量进入环境时，例如，在函数中声明一个变量，就将这个变量标记为“进入环境”。从逻辑上讲，永远不能释放进入环境的变量所占用的内存，因为只要执行流进入相应的环境，就可能会用到它们。而当变量离开环境时，则将其标记为“离开环境”。（ 2008年为止，IE，Firefox，opera，chrome，Safari的javascript都用使用了该方式）
- 引用计数：
    跟踪记录每个值被引用的次数，当声明一个变量并将一个引用类型的值赋给该变量时，这个值的引用次数就是1，如果这个值再被赋值给另一个变量，则引用次数加1。相反，如果一个变量脱离了该值的引用，则该值引用次数减1，当次数为0时，就会等待垃圾收集器的回收。

##### V8下的垃圾回收机制

V8 实现了准确式 GC，GC 算法采用了分代式垃圾回收机制。因此，V8 将内存（堆）分为新生代和老生代两部分。

###### 新生代算法

新生代中的对象一般存活时间较短，使用 Scavenge GC 算法。

在新生代空间中，内存空间分为两部分，分别为 From 空间和 To 空间。在这两个空间中，必定有一个空间是使用的，另一个空间是空闲的。新分配的对象会被放入 From 空间中，当 From 空间被占满时，新生代 GC 就会启动了。算法会检查 From 空间中存活的对象并复制到 To 空间中，如果有失活的对象就会销毁。当复制完成后将 From 空间和 To 空间互换，这样 GC 就结束了。

###### 老生代算法

老生代中的对象一般存活时间较长且数量也多，使用了两个算法，分别是标记清除算法和标记压缩算法。

在讲算法前，先来说下什么情况下对象会出现在老生代空间中：

- 新生代中的对象是否已经经历过一次 Scavenge 算法，如果经历过的话，会将对象从新生代空间移到老生代空间中。
- To 空间的对象占比大小超过 25 %。在这种情况下，为了不影响到内存分配，会将对象从新生代空间移到老生代空间中。

老生代中的空间很复杂，有如下几个空间
```
enum AllocationSpace {
  // TODO(v8:7464): Actually map this space's memory as
  read-only.
  RO_SPACE,    // 不变的对象空间
  NEW_SPACE,   // 新生代用于 GC 复制算法的空间
  OLD_SPACE,   // 老生代常驻对象空间
  CODE_SPACE,  // 老生代代码对象空间
  MAP_SPACE,   // 老生代 map 对象
  LO_SPACE,    // 老生代大空间对象
  NEW_LO_SPACE,  // 新生代大空间对象

  FIRST_SPACE = RO_SPACE,
  LAST_SPACE = NEW_LO_SPACE,
  FIRST_GROWABLE_PAGED_SPACE = OLD_SPACE,
  LAST_GROWABLE_PAGED_SPACE = MAP_SPACE
};
```

在老生代中，以下情况会先启动标记清除算法：

- 某一个空间没有分块的时候
- 空间中被对象超过一定限制
- 空间不能保证新生代中的对象移动到老生代中

这个阶段中，会遍历堆中所有的对象，然后标记活的对象，在标记完成后，销毁所有没有被标记的对象。在标记大型对内存时，可能需要几百毫秒才能完成一次标记。这就会导致一些性能上的问题。为了解决这个问题，2011 年，V8 从 stop-the-world 标记切换到增量标志。在增量标记期间，GC 将标记工作分解为更小的模块，可以让 JS 应用逻辑在模块间隙执行一会，从而不至于让应用出现停顿情况。但在 2018 年，GC 技术又有了一个重大突破，这项技术名为并发标记。该技术可以让 GC 扫描和标记对象时，同时允许 JS 运行

清除对象后会造成堆内存出现碎片的情况，当碎片超过一定限制后会启动压缩算法。在压缩过程中，将活的对象像一端移动，直到所有对象都移动完成然后清理掉不需要的内存。