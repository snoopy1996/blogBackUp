---
title: JS操作DOM优化-合并插入或文档碎片
date: 2018-07-28 20:52:48
tags: [js,dom,技术]
---

比如有一个需求，往 ul 里面添加 10 个 li

```html
<ul id="test-ul"></ul>
```

常规做法

```JavaScript
const ulNode = document.getElementById('test-ul');
for (let i = 0; i < 10; i++) {
    let iNode = document.createElement('li');
    iNode.innerHTML = i;
    ulNode.appendChild(iNode);
}
```

这里相当于操作了 10 次 DOM。

### 合并插入

```JavaScript
const ulNode = document.getElementById('test-ul');
let _html = '';
for (let i = 0; i < 10; i++) {
    _html += `<li>${i}</li>`;
}
ulNode.innerHTML = _html;
```

### 文档碎片

```JavaScript
const ulNode = document.getElementById("ul-test");
let _frag = document.createDocumentFragment();
for(let i = 0; i < 10; i++) {
    let iNode = document.createElement('li');
    iNode.innerHTML = i;
    //把元素添加进文档碎片
    _frag.appendChild(iNode);
}
//把文档碎片添加进元素
ulNode.appendChild(_frag);
```

![image](https://ws1.sinaimg.cn/large/0064OUUqly1fnyhk1f22bj30zk0m8mxy.jpg)
