---
title: AngularJs基础
date: 2018-07-28 20:55:29
tags: [AngularJs, js, 简单报告, 技术]
---

# 一、AngularJs 是什么

![](leanote://file/getImage?fileId=59dc3b4aab64415552000d3f)
AngularJS 是目前由谷歌负责开发维护的一款优秀的**跨平台（IONIC）**的**前端 JS**框架。他是为了克服 HTML 在构建应用方面的不足来设计的，换句话说，不同于 React 的拓展 JS，Angular 是拓展优化了 HTML。
同时 AngularJS 非常适合用来开发单页面 Web 应用(SPA)，得益于完整的体系结构以及完善的路由机制。
<!--more-->
目前 Angular 的最新版本是 4.0，但是由于自 Angular2.0 开始，其语法变化很大，而目前多数开发使用以及学习的都是 1.x 的版本，所以下面的代码都以 1.x 为主。

# 二、AngularJs 的特性

- MVC 结构
- 模块化开发
- 自动化双向数据绑定
- 语义化标签
- 依赖注入
- ……

---

**依赖注入**
_对于 AngularJS 来说是一个很重要的概念，几乎所有的功能都要通过依赖注入去完成对应的操作。_

## 什么是依赖注入

维基百科上的解释是：依赖注入（DependencyInjection，简称 DI）是一种软件设计模式，在这种模式下，一个或更多的依赖（或服务）被注入（或者通过引用传递）到一个独立的对象（或客户端）中，然后成为了该客户端状态的一部分。
该模式分离了客户端依赖本身行为的创建，这使得程序设计变得松耦合，并遵循了依赖反转和单一职责原则。与服务定位器模式形成直接对比的是，它允许客户端了解客户端如何使用该系统找到依赖
我们把所需要的依赖引入到项目中，但只有在用到它的时候才注入它。
**_一句话 --- 没事你不要来找我，有事我会去找你。_**
当下开发中，常用的依赖注入有 factory(工厂函数，用于返回一个对象),其他第三方的插件比如 smartTable、ngDialog、ngJsTree、vmWamp 等等。

---

# 三、AugularJs 使用基础及示例

## 1. 基本的页面结构

1.  定义 module【M】
2.  静态页面【V】
3.  定义使用控制器【C】

---

代码：

```HTML
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <!-- 引入AngularJs -->
    <script src="angular.min.js"></script>
</head>
<!-- ng-app定义应用名称，同时也是这个Web应用最底层的module -->
<!-- ng-controller用来定义一个控制器，作用范围是与之对应的module的范围 -->
<body ng-app="myApp" ng-controller="myAppCtrl">

<div>
    <h1>Hello {{name}}</h1>
</div>
<script>
	//注册应用以及控制器
    angular.module('myApp',[])
        .controller('myAppCtrl',myAppCtrl);
    //控制器实际代码
    function myAppCtrl($scope) {
        $scope.name = 'World';
    }
</script>
</body>
</html>
```

结果展示：
![](https://ws1.sinaimg.cn/large/0064OUUqly1fntv2794g2j30je06kdg8.jpg)

---

- Scope 概念：
  Scope(作用域) 是应用在 HTML (视图) 和 JavaScript (控制器)之间的纽带。
  Scope 是一个对象，有可用的方法和属性。
  Scope 可应用在视图和控制器上
  每一个 Scope 都有可作用的范围和大小，他和与之对应的 module 大小是一样的。
- 根作用域：
  所有的应用都有一个 **\$rootScope**，它可以作用在 ng-app 指令包含的所有 HTML 元素中。
  \$rootScope 可作用于整个应用中。是各个 controller 中 scope 的桥梁。用 \$rootscope 定义的值，可以在各个 controller 中使用。但**不建议**用\$rootScope 定义变量的方式去在各个 controller 间传值，如有需求，可以通过**factory**函数返回一个对象

---

## 2. 双向数据绑定

js 中的变量与通过 ng-model 指令来定义的表单内容是双向绑定的，即一个改变，另一个也会随着一起改变。
很多时候我们需要 JS 去控制 DOM 节点的内容，常规做法就是获取节点，然后通过 innerText 或者直接 value=“”的方法来操作节点。虽然 JQuery 简化了 JS 对 DOM 的操作，但是有了 Angular，则只需要一个**ng-model**指令。
示例代码：

```HTML
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>双向数据绑定</title>
    <script src="angular.min.js"></script>
</head>
<body ng-app="myApp" ng-controller="myAppCtrl">
<label>名字：</label>
<!-- 通过ng-model绑定一个变量或者一个对象的某个属性 -->
<input ng-model="user.name" placeholder="请输入你的名字">
<br>
<button ng-click="changeName()" style="margin-top: 20px"> 改变名字为Jack</button>
<pre>{{user | json}}</pre>
<script>
    angular.module('myApp',[])
        .controller('myAppCtrl',myAppCtrl);
    function myAppCtrl($scope) {
        $scope.user = {name:""};
        $scope.changeName = function () {
            $scope.user.name = 'Jack';
        }
    }
</script>
</body>
</html>
```

## 3. 重复循环 HTML

AngularJS 中有一个**ng-repeat**内置指令，用来循环 HTML 元素。

常见应用范围：

1.  表格内容显示
2.  select 选项（除 ng-repeat 外，AngularJS 还有一个专门针对 select 标签中 option 的内建指令**ng-options**）

```javaScript
function myAppCtrl($scope) {
        $scope.nameSet = ['小米','魅族','华为','中兴','OPPO'];
        var ageSet = [15,16,17,18,19,20];
        var phoneSet = ['15532105555','18832556997','18322568874','13341226635'];
        $scope.rowCollection = [];
        //向JSON数组中添加15条数据
        for (var i = 0;i<15;i++){
            var newData = {
                name:$scope.nameSet[Math.floor(Math.random()*$scope.nameSet.length)],
                age:ageSet[Math.floor(Math.random()*ageSet.length)],
                phone:phoneSet[Math.floor(Math.random()*phoneSet.length)]
            }
            $scope.rowCollection.push(newData);
        }
    }
```

表格内容：

```HTML
<tbody>
    <!-- 循环这个tr标签及其内部的所有内容 -->
    <tr ng-repeat="row in rowCollection">
    <td>{{$index+1}}</td>
    <td>{{row.name}}</td>
    <td>{{row.age}}</td>
    <td>{{row.phone}}</td>
    </tr>
</tbody>
```

select 选项

```HTML
<tfoot>
    <tr>
        <td colspan="4">
        <select style="height:35px;width:100%">
            <!-- 循环内容为数组 -->
            <option ng-repeat="row in nameSet">{{row}}</option>
        </select>
        </td>
    </tr>
</tfoot>
```

结果：
![](https://ws1.sinaimg.cn/large/0064OUUqly1fntv2quwv3j30d80fuq49.jpg)

## 4. 事件

在 AngularJs 中，通过内置指令 ng-click="functionName()"来定义单击事件，对应控制器中同名字的函数对象。

```HTML
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>单击事件</title>
    <script src="angular.min.js"></script>
</head>
<body ng-app="myApp" ng-controller="myAppCtrl">
<!-- 像JS定义单击事件那样定义AngularJs单击事件 -->
<button ng-click="clickMe()" style="margin-top: 20px"> 单击我</button>
<h2>你已单击了按钮{{clickNumber}}次</h2>
<script>
    angular.module('myApp',[])
        .controller('myAppCtrl',myAppCtrl);
    function myAppCtrl($scope) {
        $scope.clickNumber = 0;
        //对应的事件函数
        $scope.clickMe = function () {
            $scope.clickNumber++;
        }
    }
</script>
</body>
</html>
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntv35bhzgj30jn06ymxm.jpg)

另外，其他的事件还有：双击事件(ng-dbClick),隐藏(ng-hide(boolen))/显示(ng-show(boolen))。

## 5. 自定义指令(directive)

除了 AngularJS 内置的指令外，我们还可以创建自定义指令。
你可以使用 .directive 函数来添加自定义的指令。
要调用自定义指令，HTML 元素上需要添加自定义指令名。
使用驼峰法来命名一个指令， **myFirstDirective**, 但在使用它时需要以 - 分割.
另外，自定义指令共有四种可调用方式：

1.  E(Element)作为元素名使用
2.  A(Attrbute)作为属性使用
3.  C(Class)作为类名使用
4.  M 作为注释使用
    代码：

```HTML
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>自定义指令</title>
    <script src="angular.min.js"></script>
</head>
<body ng-app="myApp" ng-controller="myAppCtrl">
<div>
    <label>元素名调用：</label>
    <my-first-directive></my-first-directive>
    <label>属性调用：</label>
    <div my-first-directive></div>
    <label>类名调用：</label>
    <div class="my-first-directive"></div>
    <label>注释调用：</label>
    <!-- directive: my-first-directive -->
</div>
<script>
    angular.module('myApp',[])
        .controller('myAppCtrl',myAppCtrl)
        .directive('myFirstDirective',myFirstDirective);
    function myAppCtrl($scope) {}
    function myFirstDirective () {
        return {
            restrict : "EACM",//如不指定，默认为AE
            replace : true,//当作为注释调用的时候，必须设置此项
            template : "<h1>这里是我的第一个自定义指令!</h1>"
        };
    }
</script>
</body>
</html>
```

结果
![](leanote://file/getImage?fileId=59dc3b4aab64415552000d3e)

## 6. 内置服务

AngularJS 中预定义了很多内置服务用以满足一般前端开发所需要的情况，同时这些内置服务相比原生 JS 来说，语法上更简单，可读性更高。
以\$http 为例：
前端开发除了页面布局和与用户直接交互以外，离不开的就是和后端服务交互数据，所以必须要用到的就是 AJAX( Asynchronous JavaScript and XML | 异步的 JavaScript 和 XML)。
现将原生、JQuery、AngularJs 做一个对比。
**操作：有接口为：172.16.32.201:8080/api/allUser 执行 GET 请求。**
原生：

```JavaScript
//定义XML请求对象
var xmlhttp = new XMLHttpRequest();
//请求的回调函数
xmlhttp.onreadystatechange=function()
{
    //请求成功的操作
	if (xmlhttp.readyState==4 && xmlhttp.status==200)
	{
		console.log(xmlhttp.responseText);
	}
}
//执行请求
xmlhttp.open("GET","172.16.32.201:8080/api/allUser",true);
//发送请求
xmlhttp.send();
```

JQuery：

```JavaScript
$.ajax({
    url:'172.16.32.201:8080/api/allUser',
    type:'GET', //GET
    async:true,    //或false,是否异步
    timeout:5000,    //超时时间
    dataType:'json',    //返回的数据格式：json/xml/html/script/jsonp/text
    success:function(data,textStatus){
        console.log(data)
        console.log(textStatus)
    },
    error:function(xhr,textStatus){
        console.log('错误')
        console.log(xhr)
        console.log(textStatus)
    },
    complete:function(){
        console.log('结束')
    }
})
```

AngularJs:

```JavaScript
$http.get('172.16.32.201:8080/api/allUser')
    .success(function(res){
        console.log(res);
    })
    .error(function(err){
        console.log(err);
    });
```

**注**：AngularJS1.6 版本以上使用了 then()函数回调代替直接的 success().error(); #四、Angular 常用命令列表

## 1. 拷贝数组或对象

```javascript
angular.copy(source, [destination]);
```

## 2. 选择一个元素

```javascript
angular.element(element); //jqLite
```

## 3. 比较值是否相等

```javascript
angular.equals(o1, o2); //返回值：boolean
```

## 4. 迭代对象

```javascript
angular.forEach(obj, iterator, [context]); //iteeator是一个方法function(value,key,[obj]){处理代码}
```

## 5. 把 json 字符串转换为对象

```javascript
angular.fromJson(json);
```

## 6.把对象转换为 json 字符串

```javascript
angular.toJson(obj, pretty); //pretty为ture时，输出字符串有换行符和空格。如果设置为一个整数，JSON输出将包含许多空间每缩进（默认为2）
```

## 7. 判断是否为数组、时间、DOM 元素、函数、数字、对象、字符串、未定义、

```javascript
angular.isArray(value);
angular.isDate(value);
angular.isElement(value);
angular.isFunction(value);
angular.isNumber(value);
angular.isObject(value);
angular.isString(value);
angular.isUndefined(value);
```

# angular 指令

##常用指令

1 失去焦点事件

```html
ngBlur
```

2 值改变时触发事件

```html
<input
        ng-change="">
...
</input>
```

3 动态设置 class

```html
<input
        ng-class="">
...
</input>
```

4 点击事件

```html
<ANY
  ng-click="expression">
...
</ANY>
```

5 双击事件

```html
<ANY
  ng-dblclick="expression">
...
</ANY>
```

6 禁用元素

```html
<INPUT
  ng-disabled="expression">
...
</INPUT>
```

7 获得焦点触发事件

```html
<window, input, select, textarea, a
  ng-focus="expression">
...
</window, input, select, textarea, a>
```

8 显示、隐藏元素

```html
<ANY
        ng-hide="true">
</ANY>
<ANY
  ng-show="true">
</ANY>
```

9 判断来确定是否进行显示

```html
<ANY
  ng-if="expression">
...
</ANY>
```

10 导入其他页面

```html
<ANY
        ng-include=""
        [onload=""]
        [autoscroll=""]>
...
</ANY>
```

11 将输入文本转换为数组

```html
<input
        [ng-list=""]>
</input>
输入的文本会转换数组，默认用，分割
```

12 只读

```html
ngReadonly
```

13 遍历

```html
<div ng-repeat="(key, value) in myObj"> ... </div>
```

14 提交表单

```html
<form
        ng-submit="">
...
</form>
```

15 switch

```html
  <select ng-model="selection" ng-options="item for item in items">
  </select>
  <div class="animate-switch-container"
    ng-switch on="selection">
      <div class="animate-switch" ng-switch-when="settings|options" ng-switch-when-separator="|">Settings Div</div>
      <div class="animate-switch" ng-switch-when="home">Home Span</div>
      <div class="animate-switch" ng-switch-default>default</div>
  </div>
<!--ngSwitch指令包含ng-switch on、ng-switch-when、ng-switch-default功能类似switch，ng-switch on指要判断的值，ng-switch-when指条件条件符合将显示这个dom元素， ng-switch-default指条件都不符合默认显示的元素-->
```

## service

1.  过滤器\$filter
2.  \$http
3.  循环\$interval
4.  \$log
5.  \$q
6.  延时\$timeout
7.  \$window

## 内置过滤器

1.  格式化数字 currency
2.  格式化日期事件 date
3.  过滤数组 filter
4.  将对象转换为 json 字符串 json
5.  截取数组 limitTo
6.  转化小写大写 lowercase、uppercase
7.  格式化数字 number
8.  排序 orderBy
