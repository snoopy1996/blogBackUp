---
title: blurAdmin初探
date: 2018-07-28 21:35:28
tags: [AngularJs, blurAdmin, 技术]
---

# 基础技术要求

- HTML
- CSS 基础（多数并不需要自己完全手写样式，只是适当修改）
- 了解并可使用 margin、padding、color、line-height 等基础样式即可
- 会使用 bootstrap
- **angular.js（重点）**
  - 双向绑定
  - 依赖注入及其使用(常用：\$scope、\$http、\$tiemout、\$interval 等)
  - 路由(添加页面或改动页面路径)

以下为包含内容：**实现 CRUD 表格样式、表单样式、angular 实现 wamp 通信、Blur 框架页面的增加**。

<!--more-->

---

# CRUD 表格

## 样式效果

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntuaum7k7j31450nen2b.jpg)

## smart-table

一个表格的 js 库，内部封装了大量的常用表格操作，比如排序查找分页等。
官方文档：https://lorenzofox3.github.io/smart-table-website/
用到的属性：

- st-sort 放在表头，用于排序。官网中有两个 css 类，一定要引进项目中，否则不会出现排序时的上下黑色三角标识图标
- st-search 用于一般情况下筛选查找
- st-items-by-page 设定分页显示条目
- st-template 用于引入分页样式的模型
- st-table 基本集合(所有操作都是基于 st-table 的，排序分页查找过滤都是通过改变 st-table 的数据来实现的)
- st-safe-src 安全数据集合(异步加载刷新数据时使用)

---

    <th st-sort="userName" style="width: 10%;min-width: 80px">姓名</th>

    <th>
        <input type="text" st-search="userName" class="form-control">
    </th>

    <td colspan="7" class="text-center">
        <div st-items-by-page="10" st-pagination=""
            st-template="./lib/template/pagination.custom.html"></div>
    </td>

此处 st-template 引入的是一个已经写好的 HTML 文件，即在上图看到的分页的样式

```html
<nav ng-if="pages.length >= 2">
    <ul class="pagination">
        <li><a ng-click="selectPage(1)">首页</a>
        </li>
        <li><a ng-click="selectPage(currentPage - 1)">&lt;</a>
        </li>
        <li>
            <a>
                <page-select></page-select> of {{numPages}}
            </a>
        </li>
        <li><a ng-click="selectPage(currentPage + 1)">&gt;</a>
        </li>
        <li><a ng-click="selectPage(numPages)">尾页</a></li>
    </ul>
    </nav>
```

page-select 是通过 angular 的 directive 注册的自定义标签，实现的是通过监听输入框的内容来跳转到相应的页面的功能。源码在 src/app/them/directives/pageSelect.js 文件中。

---

**注**：一般常用搜索使用 st-search，对于有范围限制的搜索，如年龄、日期等，需要手写过滤器。原理即遍历且返回符合条件的数据。

```javascript
ng-repeat="row in displayedCollection|filter:filterdate|filter:filterMoney"//引入了两个过滤器
```

## 使用的 bootstrap 表格类

- .table
- .table-bodered
- .table-hover
- .table-condensed
- .table-responsive(**这个类要加在包裹 table 的 div 里，否则不起作用**）

注：即使加了 table-responsive，有时仍然可能因为屏幕大小导致单元格内容显示不正常，可以通过手动指定一个 min-width 来解决 ##时间显示
有时数据中传过来的时间参数是一个时间戳或者是标准日期对象，不利于直观查看，要通过手动设定一个日期格式：

```html
<td>{{row.userCreateTime | date: 'yyyy-MM-dd HH:mm:ss'}}</td>
```

前提：userCreateTime 是一个日期对象或者是一个时间戳或能与日期相互转化的字符串

---

~~## 常用插件~~

### 日期选择框——uib-datepicker-popup

#### 样式

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntubjwtd5j30af09tdgj.jpg)

#### 使用

```HTML
<div class="col-md-6">
    <p class="input-group" style="margin-bottom: 0">
        <input type="text" class="form-control" readonly
            uib-datepicker-popup="yyyy/MM/dd" ng-model="date.before"
            is-open="popup1.opened" datepicker-options="dateOptions"
            ng-required="true" close-text="Close"
            alt-input-formats="altInputFormats"/>
        <span class="input-group-btn">
            <button type="button" class="btn btn-default" ng-click="open1()">
                <i class="glyphicon glyphicon-calendar"></i>
            </button>
        </span>
    </p>
</div>
```

#### 基本属性表

<table class="table">
                        <thead>
                        <tr>
                            <th>属性</th>
                            <th>默认值</th>
                            <th>备注</th>
                        </tr>
                        </thead>
                        <tbody>
                        <tr>
                            <td>alt-input-formats</td>
                            <td>[]</td>
                            <td>手动输入日期时可接受的格式</td>
                        </tr>
                        <tr>
                            <td>clear-text</td>
                            <td>clear</td>
                            <td>清空按钮的文本</td>
                        </tr>
                        <tr>
                            <td>close-on-date-selection</td>
                            <td>true</td>
                            <td>选择一个日期后是否关闭日期面板</td>
                        </tr>
                        <tr>
                            <td>close-text</td>
                            <td>close</td>
                            <td>关闭按钮的文本</td>
                        </tr>
                        <tr>
                            <td>current-text</td>
                            <td>today</td>
                            <td>今天按钮的文本</td>
                        </tr>
                        <tr>
                            <td>datepicker-append-to-body</td>
                            <td>false</td>
                            <td>是否将日期面板放在body元素中</td>
                        </tr>
                        <tr>
                            <td>datepicker-options</td>
                            <td>[]</td>
                            <td>对Datepicker控件的配置</td>
                        </tr>
                        <tr>
                            <td>datepicker-popup-template-url</td>
                            <td>uib/template/datepickerPopup/popup.html</td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>is-open</td>
                            <td>false</td>
                            <td>是否显示日期面板</td>
                        </tr>
                        <tr>
                            <td>on-open-focus</td>
                            <td>true</td>
                            <td>打开日期面板时是否获取焦点</td>
                        </tr>
                        <tr>
                            <td>show-button-bar</td>
                            <td>true</td>
                            <td>是否在日期面板下方显示”今天”,”关闭”等按钮</td>
                        </tr>
                        <tr>
                            <td>type</td>
                            <td>text</td>
                            <td>可以改为date|datetime-local|month，这样会改变日期面板的日期格式化。</td>
                        </tr>
                        <tr>
                            <td>popup-placement</td>
                            <td>auto bottom-left</td>
                            <td>在位置前加一个auto,表示日期面板会定位在它最近一个可以滚动的父元素中.	可以设置的位置有:top top-left
                                top-right         bottom        popup-placement		bottom-left      	bottom-right      left
                                left-top       	    left-bottom       right         right-top          	right-bottom</td>
                        </tr>
                        <tr>
                            <td>uib-datepicker-popup</td>
                            <td>yyyy-MM-dd</td>
                            <td>显示日期的格式。可使用的格式见上面的列表。</td>
                        </tr>
                        </tbody>
                    </table>

---

### 消息弹窗——sweetAlert

#### 样式

![](leanote://file/getImage?fileId=595f7d3fab64414eb9001629)

```javascript
swal(
  {
    title: '', //标题内容
    text: '确定删除这条信息吗？删除之后将不能恢复信息！', //主要内容
    type: 'warning', //弹窗类型
    showCancelButton: true, //是否显示取消按钮
    confirmButtonColor: '#DD6B55', //确认按钮颜色
    confirmButtonText: '确认', //确认按钮显示内容
    cancelButtonText: '取消', //取消按钮显示内容
    closeOnConfirm: false //确认按钮关闭弹窗
    //closeOnCancel: false  //
  },
  function() {
    /*回调函数*/
  }
);

swal('已删除', '成功删除信息', 'success'); //一般常用三种提示框success/error/warning
```

#### API 使用

GitHub 地址：https://github.com/t4t5/sweetalert

---

# 表单

## 样式效果

- 添加与修改
  ![](leanote://file/getImage?fileId=595f7d3fab64414eb9001622)

## 栅格布局

表单内部 label 与 input 全部使用栅格来进行布局。
input 的 class 类为：.form-control .input-sm
详情页面的样式为给属性名外面加一个 div,然后为 div 加一个背景色，同时为所有 div 添加下边框，再稍微调整边距，可以通过修改默认样式的 CSS 或者手动添加 style 来实现。

## 模态框 ng-Dialog

ngDialog 是一个用于 Angular.js 应用的模式对话框和弹出窗口。具有轻量、易学、高度可定制等特点。

- 官方文档：http://www.bootcdn.cn/ng-dialog/readme/
  示例：
  ![](https://ws1.sinaimg.cn/large/0064OUUqly1fntuc68hf0j30dl03odfx.jpg)

```javascript
//open({})用于模态框的初始化
var dialog = ngDialog.open({
  template: 'templateId.html', //模态框模板内容
  width: 700, //宽度，可以是数值，单位是px，可以是百分比
  controller: 'moreUserModalCtrl', //控制器
  //给控制器返回一个对象：items
  resolve: {
    items: function() {
      return moreObj;
    }
  }
});
//模态框关闭之后执行的函数
dialog.closePromise.then(function(data) {
  getData();
  //data 为模态框关闭时向父级controller传递的数据
  //其中主要包含两个属性 id:唯一标识当前模态框 value: 传递回来的对象数据
  console.log(data);
});

//当使用以下方法关闭模态框的时候，data 会被传递到上面回调函数的 data.value中
//同时，如不需要传递数据，直接 $scope.closeThisDialog(); 即可
var data = {};
$scope.closeThisDialog(data);
```

# 集成 API

## 简介使用

angular 中封装了 AJAX 的操作，使得调用 API 接口与后台进行数据交互变得十分便捷。
主要由\$http+option+URL+data+success\error 回调函数组成，基本常用规范如下：

```javascript
//获取数据
$http.get(URL).then(function(res) {}, function(err) {});
//删除数据
$http.delete(URL).then(function(res) {}, function(err) {});
//添加数据
$http.post(URL, data).then(function(res) {}, function(err) {});
//修改数据
$http.put(URL, data).then(function(res) {}, function(err) {});
```

当然，并不一定按上述描述操作，但是在目前项目中，全部由上述制定操作完成前端与后端的交互，对数据的增删改查。
具体示例代码：

```javascript
$http.post('URL', data).then(
  function(res) {
    console.log(res);
  },
  function(err) {
    console.log(err);
    alert('添加失败');
  }
);
```

## 注意

在以前的项目代码或者有些老旧的教程中，会看到以下用法：

```javascript
$http
  .post('URL', data)
  .success(function(res) {
    //一些操作
  })
  .error(function(err) {
    //一些操作
  });
```

在 angular1.6 中，取消了.success()以及.error()的回调函数，如用以上写法，会碰到如下错误：
![](https://ws1.sinaimg.cn/large/0064OUUqly1fntucuiqo2j30ga06cwf7.jpg)
在 angular1.6 的后续版本中，使用.then().catch().finally()来代替 success 与 error：

```javascript
$http
  .get('http://localhost:3000/users/queryUser/node/all')
  .then(function(successCallback) {
    (function(res) {
      console.log(res);
      $scope.rowCollectionCu = res.result;
    })(successCallback.data);
  })
  .catch(function(errorCallback) {
    (function(err) {
      console.log('req ERROR');
    })(errorCallback.data);
  });
```

注：在当前项目中，指定的版本为 1.5.8，所以仍旧可以使用 success 与 error 函数，但不建议。

# angular-wamp

实时通信：

A 每一秒向服务器发送一个主题为“数字”，内容分别为“1”，“2”，“3”……“10”。
B 从第一秒连接到服务器，并且订阅了主题为“数字”的消息，到第 10 秒，他收到了 1-10 的数字，每一秒收到一条。
C 从第 5 秒连接到服务器，同样订阅了主题为“数字”的消息，到第 10 秒，他收到了 6-10 的数字，同样的每一秒收到一条。

效果：
![](https://ws1.sinaimg.cn/large/0064OUUqly1fntudjcxz4j31450hj776.jpg)
在 angular 中实现 wamp 通信可以使用**angular-wamp**插件库。

### 1. 安装：`bower install angular-wamp --save`

### 2. 注入：'vxWamp'

### 3. 配置初始化：

```javascript
function wampConfig($wampProvider) {
  $wampProvider.init({
    url: 'ws://ip:port/ws', //服务的IP端口
    realm: 'realm1' //认证信息
  });
}
```

### 4. 开启服务：

```javascript
 .run(function($wamp){
            $wamp.open();//程序一运行,启动wamp服务
 });
```

### 5. 收发消息

```javascript
//接收wamp消息的回调函数
function onevent(args) {
  var hello = args[0];
  console.log(hello);
  hello.time = new Date();
}
//订阅主题为"example"的消息
$wamp.subscribe('example', onevent);

//每一秒发送一条hello world的主题消息到服务器，并打印已经发送
$interval(function() {
  $wamp.publish('com.myapp.hello', ['hello world!']);
  console.log('已经发送');
}, 1000);
```

# blur 框架

## 运行项目

```
npm install -g gulp
npm install
bower install
gulp serve
```

**Node 版本 6 ~ <7**

由于某些网络因素，导致 npm 使用比较‘艰难’：npm 的服务器是在国外的，而国内访问国外的服务要么很慢，要么根本不能使用。

解决办法：

1.  使用淘宝提供的 cnpm 安装 https://npm.taobao.org/ (使用 cnpm 下载的依赖是很多 @ 开头的链式依赖)
2.  直接更换 npm 的源

```
npm config set registry https://registry.npm.taobao.org
npm config list
```

另外 ，还有一个比较好用的工具可以方便的切换 npm 源：nrm

---

## gulp(暂时忽略)

### gulp 简介

Gulp 是基于 Node.js 的一个构建工具（自动任务运行器），开发者可以使用它构建自动化工作流程（前端集成开发环境）。一些常见、重复的任务，例如：网页自动刷新、CSS 预处理、代码检测、压缩图片、等等…… 只需用简单的命令就能全部完成。
###gulp 使用(非必须，了解即可）
**有需要时在进行学习**
Gulp 的核心 API 只有 4 个：src、dest、task、watch

- gulp.src(globs[, options])：指明源文件路径
  globs：路径模式匹配；
  options：可选参数；
- gulp.dest(path[, options])：指明处理后的文件输出路径
  path：路径（一个任务可以有多个输出路径）；
  options：可选参数；
- gulp.task(name[, deps], fn)：注册任务
  name：任务名称（通过 gulp name 来执行这个任务）；
  deps：可选的数组，在本任务运行中所需要所依赖的其他任务（当前任务在依赖任务执行完毕后才会执行）；
  fn：任务函数（function 方法）；
- gulp.watch(glob [, opts], tasks)：监视文件的变化并运行相应的任务
  glob：路径模式匹配；
  opts：可以选配置对象；
  taks：执行的任务；

gulp 简介使用：http://www.bluesdream.com/blog/gulp-frontend-automation-plugins-instructions.html

示例：压缩编译\*.html

```javascript
gulp.task('html', function() {
  return gulp
    .src('src/**/*.html') // 指明源文件路径、并进行文件匹配
    .pipe('dist'); // 输出路径
});
```

执行编译任务：`gulp html`

---

## 页面结构

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntun5tjzuj30p60hwwml.jpg)

---

## 页面跳转

路由是单页面应用中“页面跳转”的必要组成部分。
Angular 有自己的内置路由模块，叫做 ngRouter，但是，由于其功能局限性（主要体现在多层嵌套中）,当前使用的是 ui-router，一个基于 ngRouter 开发的第三方路由模块，同时他也是 Blur 框架原生使用的路由模块。具体的二者的比较可以参考下面一篇文章。
Angular 路由深入浅出：http://www.tuicool.com/articles/MjIR3a

---

## 添加页面

通过对 module 的子异己模块的添加来实现实际页面的增加，同时通过 state 来实现跳转路径的注册。

示例：在导航栏中添加一个一级菜单，叫做测试 1，然后为其添加一个页面，叫做路由测试。
当前结构
![](leanote://file/getImage?fileId=595f7d3fab64414eb900162a)

1.  在 pages.module.js 中添加一个 module 为 test:
    ![](leanote://file/getImage?fileId=595f7d3fab64414eb9001628)
2.  创建一个 test 文件夹，然后添加一个 test.module.js，然后里面再加一个 test1 文件夹，添加 test1.html 与 testCtrl.js 文件
    ![](leanote://file/getImage?fileId=595f7d3fab64414eb9001623)
    结果
    ![](leanote://file/getImage?fileId=595f7d3eab64414eb900161f)

---

# 备注

## 关于整体

- Blur 原生框架可在 GitHub 上搜索“BlurAdmin”下载。注意注释掉 index.html 页面的谷歌地图
- 考虑到压缩包大小，已经将 node_modules 文件夹与 bower_components 文件夹去掉，可以通过 npm install 与 bower install 安装依赖
- 所发的代码并非是演示时的代码，做了一些调整
- dashboard 主页面为高德地图，有兴趣有时间的可以自己实现一下
- 主要的 js 文件都有详细注释，不懂得先看注释。
- 后期手动添加的样式在 src/sass/main.scss 文件中

## 关于 api

- 关于代码中所用 API 为我用 node 与 mongo，然后配合 Express，router 简单实现的对数据库的增删改查，没有用 rest，只是为了做个使用 API 的示范。
- 如果已经在本地安装了 node 与 mongo，可以先运行 mongoTest 项目，再运行 Blur。
- 如果没有安装则可以 src/app/pages/business/customer/customerCtrl.js 文件中注释掉所有 getData()方法，直接运行 Blur，只是不再具备增删改查的实际功能。或者也可以找学习后端的同学帮忙做几个接口，相互配合。
- API 的引用是通过 APPConfig 注入的，appConfig 是一个 factory，在 app.js 文件中有声明，用于返回 config 对象。

## 关于结构

- 所有的页面显示全部在 src 目录下的 app 文件夹。
- 最底层的 module 注入，在 app.js 文件中
- 显示页面在 pages.module.js 文件中
- 自定义组件，比如 pageToo 在 them 文件夹中

## 关于 wamp

- wamp 的配置与连接运行都是在 app.js 文件中。
- wamp 的使用是在 src/app/pages/business/employeeCtrl.js 文件中。
- 至于具体实现方面，可以找一下已经能运行 wamp 服务并且能够发送主题的消息的同学配合，将 app.js 中的 wamp 配置 IP 与认证连接信息换成相应的 IP 地址与参数信息。或者写一个随机函数实现模拟效果（掌握 angular-wamp 的使用)

## 关于页面

- 除上述演示方法外，同样可以使用为每一个内容页单独指定 module 的方法来实现，具体可看 src/app/pages/business 文件夹中页面的实现，与 test 相互对照。
- login 页面是以 base.html 为父级页面的，查看效果可以通过右上角头像的退出登陆来跳转到不含导航栏与侧边栏的登录页。
