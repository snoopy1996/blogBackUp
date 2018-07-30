---
title: JS对象数组排序
date: 2018-07-28 20:53:31
tags: [js,排序,技术]
---

V8 引擎内置了排序方法 `sort()`,用于对数组的元素进行排序,并返回数组。默认排序顺序是根据字符串 Unicode 码点。

但是更多时候，我们需要更加专注的排序，比如根据对象数组里的某个属性排序。

`sort`方法接收一个函数作为参数，这里嵌套一层函数用来接收对象属性名，其他部分代码与正常使用 sort 方法相同。

<!--more-->

```JavaScript
var arr = [
    {name:'zopp',age:0},
    {name:'gpp',age:18},
    {name:'yjj',age:8}
];

function compare(property){
    return function(a,b){
        var value1 = a[property];
        var value2 = b[property];
        return value1 - value2;
    }
}
console.log(arr.sort(compare('age')))
```

那，如果想要指定排序方式：升序 or 降序：

```JavaScript
sortBy: function(attr,rev){
    // 第二个参数没有传递 默认升序排列
    if(rev ==  undefined){
        rev = 1;
    }else{
        rev = (rev) ? 1 : -1;
    }
    return function(a,b){
        a = a[attr];
        b = b[attr];
        if(a < b){
            return rev * -1;
        }
        if(a > b){
            return rev * 1;
        }
        return 0;
    }
}
```

使用方法：

```JavaScript
newArray.sort(sortBy('number',false));
```

![image](https://ws1.sinaimg.cn/large/0064OUUqly1fnyhk1f22bj30zk0m8mxy.jpg)
