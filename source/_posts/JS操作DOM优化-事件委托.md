---
title: JS操作DOM优化-事件委托
date: 2018-07-28 20:51:46
tags: [js, dom, 技术]
---

### What

> 事件委托，通俗地来讲，就是把一个元素响应事件（click、keydown......）的函数委托到另一个元素；

> 一般来讲，会把一个或者一组元素的事件委托到它的父层或者更外层元素上，真正绑定事件的是外层元素，当事件响应到需要绑定的元素上时，会通过事件冒泡机制从而触发它的外层元素的绑定事件上，然后在外层元素上去执行函数。

### Why

在 JavaScript 编程中，经常会遇到 JS 操作 DOM 绑定特定事件的情况，在这种情况下需要利用 JS 的`document`内置对象去获取元素并做相应的修改。

但是，当这种修改设计大量的 DOM 或者次数变得频繁且多的时候，会引起浏览器不断的进行重绘或重排，影响页面响应与交互，严重的时候，甚至引起浏览器的卡死。

所以，需要一种方式来简化或者减少 JS 对 DOM 的操作，但又要保证对需求的实现。

举个例子，比如宿舍里每个人在同一时间都有快递需要去快递点签收，于是，宿舍所有人全部都去了快递点，这显然是不合常理的，最符合逻辑的做法是，每个人把自己需要签收的快递告诉同一个人，然后由这个人取快递点为所有代收快递，这就是委托。

### How

在浏览器中，dom 层级的事件是按顺序依次向外执行的，也就是我们通常所讲的`事件冒泡`。

比如，有一个 dom 层级是：

```html
<div>
    <ul>
        <li>
            <a>Click Me</a>
        </li>
    </ul>
</div>
```

如果，我们点击了 a 元素，那么，这个`Click`事件会沿`a > li > ul > div` 顺序依次执行，也就是，如果此时 div 加了事件监听，那么这个事件会被触发。

利用这个原理，我们可以为一类事件做一个委托。把事件具体内容放到他们共同的父级容器上。这样，只需要一次 dom 操作，便可完成一类操作。

例如：

```html
<ul id="click-ul-test">
    <li>1</li>
    <li>1</li>
    <li>1</li>
    <li>1</li>
    <li>1</li>
    <li>1</li>
    <li>1</li>
    <li>1</li>
    <li>1</li>
    <li>1</li>
</ul>
```

**需求：**点击 li 标签的时候，输出`hello world`

常规做法：

```JavaScript
let UlNode = document.getElementById('click-ul-test');
let LiNodes = UlNode.getElementByTagName('li');
for(let i = 0, len = LiNodes.length; i < len; i++) {
    LiNodes[i].addEventListener('Click', () => {
        alert('hello world');
    });
}
```

这种做法的优点就是简洁明了，但是，这是 10 个 li 标签，就需要 10 次操作。如果 li 标签有成百上千个的时候，岂不是需要操作成百上千次 dom。

另外，如果里面含有特定操作下后来插入的 li 标签，那么这个写法还怎么保证事件能正确被监听？

所以，事件委托的做法：

```JavaScript
let UlNode = document.getElementById('click-ul-test');
UlNode.addEventListener('Click', () => {
    alert('hello world');
});
```

这样，只需要一次 DOM 操作，所有 li 的点击事件都会触发`alert(''hello world)`。

那么，如果`ul`标签有个比较大的上下内边距，在点击 li 标签以外的区域的时候也会触发事件，又该如何？

这时候，我们需要利用 `target` 属性来进一步确认 click 的源头标签元素名称，来决定是否进行实际的逻辑触发:

```JavaScript
let UlNode = document.getElementById('click-ul-test');
UlNode.addEventListener('Click', (e) => {
    let ev = e.target || ev.srcElement;
    if (e.tagName.toLowerCase() === 'li') {
        alert('hello world');
    }
});
```

### 拓展

- 如果给这些标签加不同的点击事件呢？

```html
<div id="box">
    <input type="button" id="add" value="添加" />
    <input type="button" id="remove" value="删除" />
    <input type="button" id="move" value="移动" />
    <input type="button" id="select" value="选择" />
</div>
```

```JavaScript
var oBox = document.getElementById("box");
oBox.onclick = function (ev) {
    var ev = ev || window.event;
    var target = ev.target || ev.srcElement;
    if(target.nodeName.toLocaleLowerCase() === 'input'){
        switch(target.id) {
            case 'add' :
                alert('添加');
                break;
            case 'remove' :
                alert('删除');
                break;
            case 'move' :
                alert('移动');
                break;
            case 'select' :
                alert('选择');
                break;
        }
    }
}
```

- 如果`li`标签内部还有其他标签，且这些标签随机出现又不固定，那么 `target` 出现的可能是 `li` ，也可能是其他诸如 `div`、`a`等标签，又该如何？

```html
<ul id="test">
    <li>
        <p>11111111111</p>
    </li>
    <li>
        <div>2222222</div>
    </li>
    <li>
        <span>3333333333</span>
    </li>
    <li>4444444</li>
</ul>
```

tip: 可以使用一个循环向上查找父级 node，如果有 li，则触发事件。

```JavaScript
var oUl = document.getElementById('test');
oUl.addEventListener('click',function(ev){
    var target = ev.target;
    while(target !== oUl ){
        if(target.tagName.toLowerCase() === 'li'){
            console.log('li click~');
            break;
        }
        target = target.parentNode;
    }
});
```

![image](https://ws1.sinaimg.cn/large/0064OUUqly1fnyhk1f22bj30zk0m8mxy.jpg)
