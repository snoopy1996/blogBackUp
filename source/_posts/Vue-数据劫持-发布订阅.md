---
title: Vue-数据劫持+发布订阅
date: 2018-08-22 13:16:15
tags: [技术, Vue, 数据劫持, 发布订阅]
---

> Vue (读音 /vjuː/，类似于 view) 是一套用于构建用户界面的渐进式框架。

`Vue`的数据绑定机制不同于`AngularJs`的脏检查机制，而是基于`Object.defineProperty()`来实现的，通过这个函数可以监测到`set`和`get`的事件。

<!-- more -->

#### 简单尝试

```javascript
var obj = {};
Object.defineProperty(obj, 'text', {
  /**
   * 返回值就是通过 object.propName 获取的结果值
   * 如果这里改了返回值，那么调用的时候，就会按改的来取值
   */
  get: function() {
    console.log('调用了 get 获取 text 属性');
    return '哈哈，我已经改了值，你获取不到内容的';
  },
  set: function(val) {
    console.log('调用了set 更改text ->', val);
  }
});

obj.text = 'this is my Object.defineProperty test';
console.log(obj.text);
```

![result](https://ws1.sinaimg.cn/large/0064OUUqly1fuib65a5zoj30et020glh.jpg)

> get 和 set 方法内部的 this 都指向 obj，这意味着 get 和 set 函数可以操作对象内部的值。另外，访问器属性的会"覆盖"同名的普通属性，因为访问器属性会被优先访问，与其同名的普通属性则会被忽略（也就是所谓的被"劫持"了）

#### 简单的绑定实现

```html
<input id="input-test" />
<span id="span-test"></span>
<button onclick="changeText()">改对象的值</button>
```

```javascript
var obj = {};
Object.defineProperty(obj, 'text', {
  set: function(val) {
    console.log('调用了set 更改text ->', val);
    document.getElementById('input-test').value = val;
    document.getElementById('span-test').innerText = val;
  }
});

document.getElementById('input-test').addEventListener('keyup', function(e) {
  obj.text = e.target.value;
});

function changeText() {
  obj.text = '按钮改了对象的值';
}
```

![直接输入](https://ws1.sinaimg.cn/large/0064OUUqly1fuib9ufgcuj3099016jr7.jpg)
![点击按钮](https://ws1.sinaimg.cn/large/0064OUUqly1fuiba6i39lj30cs01cmx1.jpg)

---

以上就是 Vue 实现双向绑定的基本原理，当然，是一个完全特定的实现：特定表单、特定 span，特定对象。

所以，对于 Vue 而言，还缺了一个模板语法解析以及动态绑定的功能。

Vue 中是：

```html
<div id="app">
    <input type="text" v-model="text" />
    <span>{{text}}</span>
</div>
```

```javascript
var vm = new Vue({
  el: '#app',
  data: function() {
    return {
      text: 'hello world'
    };
  }
});
```

所以，大致应该把初始化过程分为以下几个过程：

1. 表单中 model 以及节点双括号的解析绑定
2. 输入框内容变化时，实际数据对象的变化以及对应其他视图的变化
3. 实际数据对象变更时，与之关联的所有视图的更新

#### 简单的发布者、订阅者模式

```javascript
// 订阅者s
var sub1 = {
  update: function() {
    console.log(1);
  }
};
var sub2 = {
  update: function() {
    console.log(2);
  }
};
var sub3 = {
  update: function() {
    console.log(3);
  }
};

function Dep() {
  this.subs = [sub1, sub2, sub3];
}
// 收到发布消息
Dep.prototype.notify = function() {
  this.subs.forEach(function(sub) {
    sub.update();
  });
};

var dep = new Dep();
// 发布者
var pub = {
  publish: function() {
    dep.notify();
  }
};
// 发布消息
pub.publish(); // 1 2 3
```

---

#### 完整实现

```html
<div id="app">
    <input type="text" v-model="text" />
    <span>{{text}}</span>
</div>
```

```javascript
var obj = {};
Object.defineProperty(obj, 'text', {
  set: function(val) {
    console.log('调用了set 更改text ->', val);
  }
});
// dependence
function Dep() {
  this.subs = [];
}
// 添加原型方法
Dep.prototype = {
  addSub: function(sub) {
    this.subs.push(sub);
  },
  // 通知
  notify: function() {
    this.subs.forEach(function(sub) {
      sub.update();
    });
  }
};
// 观察者：收到通知之后更新视图
function Watcher(vm, node, name) {
  Dep.target = this;
  this.name = name;
  this.node = node;
  this.vm = vm;
  this.update();
  Dep.target = null;
}
// 原型方法
Watcher.prototype = {
  update: function() {
    this.get();
    // 更改 {{}} 内容
    this.node.innerText = this.value; // 这里会调用 get
  },
  // 返回当前改变的值
  get: function() {
    this.value = this.vm[this.name];
  }
};

// 初始化赋值替换
function compileNode(node, vm) {
  var reg = /\{\{(.*)\}\}/;

  if (node.nodeType === 1) {
    var attrList = node.attributes;
    // 节点类型，查找 v-model 进行替换
    for (var i = 0, max = attrList.length; i < max; i++) {
      if (attrList[i].nodeName === 'v-model') {
        var name = attrList[i].nodeValue;
        // 绑定输入事件监听
        node.addEventListener('input', function(e) {
          vm[name] = e.target.value; // 更改数据模型
        });
        node.value = vm[name];
        node.removeAttribute('v-model');
      }
    }
    // 是否是双括号绑定数据
    if (reg.test(node.innerText)) {
      var name = RegExp.$1; // 获取name
      name = name.trim(); // 去除无用空字符
      node.innerText = vm[name]; //赋值
      new Watcher(vm, node, name); // 生成一个观察者实例
      // console.log(name, vm[name]);
    }
  }
}

// 返回替换后的dom
function nodeToFragment(node, vm) {
  var flag = document.createDocumentFragment();
  var child;

  while ((child = node.firstChild)) {
    compileNode(child, vm);
    flag.append(child); // 劫持node的所有子节点
  }

  return flag;
}

function defineReactive(obj, key, val) {
  var dep = new Dep();
  Object.defineProperty(obj, key, {
    get: function() {
      // 如果在取值的时候，Dep.target存在，则将其加入订阅者列表
      if (Dep.target) {
        dep.addSub(Dep.target);
      }
      return val;
    },
    set: function(newVal) {
      if (newVal === val) {
        return;
      }
      val = newVal;
      // 发布消息：值改变
      dep.notify();
      console.log('key->', val);
    }
  });
}
// 为所有属性增加默认事件，重写setter getter
function observe(obj, vm) {
  Object.keys(obj).forEach(function(key) {
    defineReactive(vm, key, obj[key]);
  });
}
// Vue 构造函数
function Vue(options) {
  this.data = options.data();

  var data = this.data;
  observe(data, this);

  var id = options.el;
  var dom = nodeToFragment(document.getElementById(id), this);
  document.getElementById(id).appendChild(dom);
}

var vm = new Vue({
  el: 'app',
  data: function() {
    return {
      text: 'hello world'
    };
  }
});
```

![最终效果](https://ws1.sinaimg.cn/large/0064OUUqly1fuietyzx87j30as01j3ya.jpg)

在线预览测试地址：[supersy.xyz/vue-ODP](http://supersy.xyz/vue-ODP/)(可直接赋值源码本地测试)
